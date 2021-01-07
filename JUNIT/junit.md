# SpringBoot 1.4.15 集成 JUnit 4

## IDEA 安装插件

- FILE --> SETTING --> PLUGINS --> Marketplace

- 搜索junit, 找JunitGenerator V2.0 安装

## 写测试代码

### controller
```java
@Slf4j
@RestController
@RequestMapping("validate")
public class ValidateController {
    @GetMapping("/prime")
    public String isPrime(@RequestParam("number") Integer number) {
        return number % 2 == 0 ? "Even" : "Odd";
    }

    @PostMapping("/add-user")
    public ResponseData addUser(@RequestBody User user){
        return ResponseUtil.successResponse(user);
    }
}

```

### 生成Test类

- 安装插件后, 在Controller类中, 鼠标光标放在ValidateController上, 按快捷键 ALT + INSERT

- 选junit test --> junit4

- 将生成的ValidateControllerTest类从java包中转移到test包下, 创建相同的package路径

### Test类编码
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = DemoContractProviderApplication.class)
@ActiveProfiles("test") // 指定当前的test使用哪个配置文件, 配置文件为test包下的yml配置文件
public class ValidateControllerTest {

    private MockMvc mockMvc;

    @Resource
    private WebApplicationContext webApplicationContext;

    @Before
    public void before() throws Exception {
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }

    @After
    public void after() throws Exception {
    }

    @Test
    public void testIsPrime() throws Exception {
        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.get("/validate/prime?number=2")).andReturn();
        int status = mvcResult.getResponse().getStatus();
        Assert.assertEquals("请求错误", 200, status);
        String responseString = mvcResult.getResponse().getContentAsString();
        Assert.assertEquals("返回结果不一致", "Even", responseString);

        mvcResult = mockMvc.perform(MockMvcRequestBuilders.get("/validate/prime?number=3")).andReturn();
        status = mvcResult.getResponse().getStatus();
        Assert.assertEquals("请求错误", 200, status);
        responseString = mvcResult.getResponse().getContentAsString();
        Assert.assertEquals("返回结果不一致", "Odd", responseString);
    }

    
    @Test
    public void testAddUser() throws Exception {
        String path = "/validate/add-user";
        User user = User.builder().userId(1234L).userName("zhangsan").build();

        ObjectMapper mapper = new ObjectMapper();
        // user 对象转json,
        String json = mapper.writeValueAsString(user);
        // 如果能直接提供json, 这个地方可以直接写json
        // String json = "{\"userId\":1234,\"userName\":\"zhangsan\"}";

        // 由于各接口增加了权限校验的逻辑, 所以模拟请求中增加header
        // 需要用户中心配置一个权限比较全的用户.
        HttpHeaders headers = new HttpHeaders();
        headers.add("jtpf.userId", "1234");
        headers.add("jtpf.sysCode", "1234");
        // 投保的接口需要增加一个token, 来校验是否重复投保
        headers.add("X-RESUB-TOKEN", "1231242135235");

        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders
                        .post(path)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(json)
                        .headers(headers)).andReturn();

        int status = mvcResult.getResponse().getStatus();
        Assert.assertEquals("请求错误", 200, status);

        String responseString = mvcResult.getResponse().getContentAsString();
        ResponseData response = mapper.readValue(responseString, ResponseData.class);
        log.info("responseString:{}", responseString);
        Assert.assertEquals("请求失败", "0000", response.getCode());
    }
}

```










## 引用文章

### 单元测试实践（SpringCloud+Junit5+Mockito+DataMocker）
```
https://www.cnblogs.com/pluto4596/p/11703382.html
```

### SpringBoot基本操作——SpringBoot使用Junit4单元测试（有demo）
```
https://blog.csdn.net/zhulier1124/article/details/82228831
```
