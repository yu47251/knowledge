# RestFull 问题
- put代表更新资源, 整体替换. 在Spring中提供的RestTemplate提供的put方法的调用是没有返回值的
- delete代表删除资源, 与put一致, 也没有返回值.
```java
        RestTemplate restTemplate = new RestTemplate();
        String body = "[{\"applicationFormCode\":\"TB201904231920337890\",\"polEndDate\":\"2049-04-25 02:05:48\",\"polStartDate\":\"2019-04-25 02:05:48\"}]";
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
        restTemplate.put("http://10.10.206.215:6032/applicationforms/duration", body);
```
- 为了解决没有返回值问题, 提供以下解决方案, 采用exchange代替put方法, 在参数中传入put类型:
```java
        RestTemplate restTemplate = new RestTemplate();
        String body = "[{\"applicationFormCode\":\"TB201904231920337890\",\"polEndDate\":\"2049-04-25 02:05:48\",\"polStartDate\":\"2019-04-25 02:05:48\"}]";
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
        HttpEntity<String> entity = new HttpEntity<>(body, headers);
        ResponseEntity<String> result = restTemplate.exchange("http://10.10.206.215:6032/applicationforms/duration", HttpMethod.PUT, entity, String.class);
        System.out.println( result.getBody());
```