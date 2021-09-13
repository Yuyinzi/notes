项目中为了接收来自设备的告警信息,由于不同类别的告警信息结构不固定,所以采用`MongoDB`进行存储.
## 建立collection
`collection`在`mongo`中类似于关系型数据库的`table`.
```
// pom.xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```
指定`mongo`地址:
```
spring.data.mongodb.host = 192.168.12.86:30017
spring.data.mongodb.database = vb
```
设置一个`ApplicationRunner`让项目启动时检查是否存在该`collection`:
```
@Component
public class InitMongoApplicationRunner implements ApplicationRunner {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("检查mongo...");
        if (!mongoTemplate.collectionExists("testdb")) {
            System.out.println("生成告警表...");
            mongoTemplate.createCollection("testdb");
        }
    }
}
```
使用终端查看是否创建成功:
```
[~] mongo 192.168.12.86:30017/vb 
> db.getCollectionNames();
```
> 如果使用的`mongo shell`是`2.x`,返回会为空.
## 插入数据
```
@Service
@Slf4j
public class TestMongoServiceImpl implements TestMongoService {
    @Autowired
    MongoTemplate mongoTemplate;

    @Override
    public void insertAlertMsg(AlertMsgEntity alertMsgEntity) {
        String collectionName = "testdb";
        mongoTemplate.insert(alertMsgEntity, collectionName);
        log.info("插入告警完成");
    }
}
```
其中`AlertMsgEntity`的定义为:
```
// AlertMsgEntity.java
@Data
public class AlertMsgEntity {
    private String method;
    private AlertParamsEntity params;
}
// AlertParamsEntity.java
@Data
public class AlertParamsEntity {
    private String sendTime;
    private String ability;
    private List<AlertEventEntity> events;
}
```
使用`postman`,往简单的`controller`里插入数据:
```
    @PostMapping(value="/eventRcv")
    public BaseResponseVo<String> eventReceive(@RequestBody AlertMsgEntity alertMsgEntity){
        TestMongoService.insertAlertMsg(alertMsgEntity);
        return BaseResponseVo.ok();
    }
```
插入几条如下格式的`json`:
```
{
	"method": "OnEvent",
	"params": {
		"ability": "rule",
		"events": [{
			"data": {
				"dataType": "behavior",
				"recvTime": "2017-04-22T15:39:01+08:00",
				"sendTime": "2017-04-22T15:39:01+08:00",
				"dateTime": "2017-04-22T15:39:01+08:00",
				"ipAddress": "10.10.10.10",
				"portNo": 80,
				"channelID": 1,
				"eventType": "detection",
				"eventDescription": "detection",
				"detection": [{
					"targetAttrs": "",
					"imageUrl": "",
					"direction": 1,
					"sensitivityLevel": 20,
					"planeHeight": 30,
					"detectionTarget": 1,
					"regionCoordinatesList": [{
							"positionX": 1,
							"positionY": 1
						},
						{
							"positionX": 1,
							"positionY": 1
						}
					]
				}]
			},
			"eventId": "12334567",
			"eventType": 2,
			"happenTime": "2021-08-09T15:17:24.000+08:00",
			"srcIndex": "1",
			"srcName": "",
			"srcType": "",
			"status": 0,
			"timeout": 0
		}],
		"sendTime": "2019-01-02T15:19:59.857+08:00"
	}
}
```
查询所有记录:
```
> db.testdb.find();
```
查询指定`eventType`的记录:
```
> db.testdb.find({'params.events.eventType':2});
```
在此基础上查询日期大于当天的记录:
```
> db.testdb.find({'params.events.eventType':2, 'params.events.happenTime':{$gt:'2021-08-09'}});
```
使用`distinct`去除重复的告警记录:
```
db.testdb.distinct('params.events.eventId',{'params.events.eventType':2, 'params.events.happenTime':{$gt:'2021-08-09'}});
```
对应查询今天告警次数的操作的`java`代码:
```
    public Integer lineDetectionMsgCount() {
        LocalDate today = LocalDate.now();
        String BEGIN_DAY_TIME_FORMAT = "yyyy-MM-dd 00:00:00";
        String todayStr = today.format(DateTimeFormatter.ofPattern(BEGIN_DAY_TIME_FORMAT));
        Criteria criteria = Criteria.where("params.events.eventType")
            .is(2).and("params.events.happenTime").gte(todayStr);
        Query query = new Query(criteria);
        int count = mongoTemplate.findDistinct(query, "params.events.eventId", "testdb", AlertMsgEntity.class).size();
        return count;
    }
```
### 建索引
一种方案是在建表的时候建立索引:
```
mongoTemplate.getCollection(collectionName).createIndex(
               new Document("happenTime", "hashed"), new IndexOptions().background(false).name("happen_time_idx")
           );
```
另一种方案是,使用`@Document` 建表,利用`@Indexed`建索引,不需要再在`ApplicationRunner`中进行初始化了:
```
@Data
@Document(collection = "testdb")
public class AlertMsgDto {

    private String method;
    private String ability;
    private String sendTime;
    private String eventId;
    private String srcIndex;
    private String srcType;
    private String srcName;
    @Indexed(name="event_type_idx")
    private Integer eventType;
    private Integer status;
    private Integer eventLvl;
    private Integer timeout;
    @Indexed(name="happen_time_idx")
    private String happenTime;
    private String srcParentIndex;
    private Object data;
}
```
使用`db.testdb.getIndexes()`可以查看索引是否建立成功.