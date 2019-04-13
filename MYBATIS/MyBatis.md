# MyBatis 

在开发过程中，遇到了1对多的情况，进行级联查询并封装，代码记录：

一个TProductMd对应多个TBizEntryDef
## mapper的写法
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="jtpf.ins.svc.mapper.TProductMdMapper" >
  <resultMap id="BaseResultMap" type="jtpf.ins.svc.entity.TProductMd" >
    <id column="id" property="id" jdbcType="INTEGER" />
    <result column="product_code" property="productCode" jdbcType="VARCHAR" />
    ...
    <result column="channel_code" property="channelCode" jdbcType="VARCHAR" />
  </resultMap>

  <resultMap id="ResultMapWithEntry" type="jtpf.ins.svc.entity.vo.TestVO" extends="BaseResultMap">
    <collection property="entrys" ofType="jtpf.ins.svc.entity.TBizEntryDef">
      <id column="bed_id" property="id" jdbcType="INTEGER" />
      ...
      <result column="project_name" property="projectName" jdbcType="VARCHAR" />
    </collection>
  </resultMap>

  <select id="selectProductWithList" resultMap="ResultMapWithEntry">
    SELECT
        pm.id,
        ...
        bed.project_name 
    FROM
        t_product_md pm
        LEFT JOIN t_biz_entry_def bed ON pm.channel_code = pm.channel_code 
        AND bed.channel_code IS NOT NULL 
    ORDER BY
        product_code
  </select>
</mapper>
```
## 实体类的结构
- vo采用lombok的方式初始化get，set方法；对应DB的实体，使用generator生成的实体类
```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class TestVO extends TProductMd {
    private List<TBizEntryDef> entrys;
}


@Table(name = "t_product_md")
public class TProductMd {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
        ...
}


@Table(name = "t_biz_entry_def")
public class TBizEntryDef {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
        ...
}
```