---
title: "Mybatis TypeHandler 使用案例"
date: 2019-09-09T07:20:41+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

数据库的数据类型与 Java 中的数据类型存在差异，当需要对两者做特定的转换时，Mybatis 提供了 TypeHandler 来实现转换操作，以下是一个实际案例，通过将 Java 中的 List 转换成 String 进行存储，在取出来的时候将 String 还原成 List。

# 1 实现一个 TypeHandler 处理器

```java
@MappedJdbcTypes(JdbcType.VARCHAR)
@MappedTypes(List.class)
public class ListToStringTypeHandler implements TypeHandler {

    private static final String LIST_SPLIT_FLAG = ",";

    @Override
    public void setParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
        List<Integer> list = (List<Integer>) parameter;
        StringBuilder stringBuilder = new StringBuilder();
        if (CollectionUtil.isNotEmpty(list)) {
            int j = 1;
            for (Integer num : list) {
                stringBuilder.append(num);
                if (j++ < list.size()) {
                    stringBuilder.append(",");
                }
            }
        }
        ps.setString(i, stringBuilder.toString());
    }

    @Override
    public Object getResult(ResultSet rs, String columnName) throws SQLException {
        String str = rs.getString(columnName);
        if (StringUtils.isNullOrEmpty(str)) {
            return null;
        }
        return CollectionUtil.newArrayList(str.split(LIST_SPLIT_FLAG));
    }

    @Override
    public Object getResult(ResultSet rs, int columnIndex) throws SQLException {
        String str = rs.getString(columnIndex);
        return CollectionUtil.newArrayList(str.split(LIST_SPLIT_FLAG));
    }

    @Override
    public Object getResult(CallableStatement cs, int columnIndex) throws SQLException {
        String str = cs.getString(columnIndex);
        return CollectionUtil.newArrayList(str.split(LIST_SPLIT_FLAG));
    }
}
```

# 2 在需要使用的地方进行调用

```xml
    <resultMap type="dept" id="deptMap">
        <result column="idList" property="idList" typeHandler="me.yuanzx.type.handler.demo.ListToStringTypeHandler"/>
    </resultMap>

    <insert id="deptSave">
      insert into dept
      values(#{deptNo},#{dname},#{loc},#{flag},
      #{idList, typeHandler=me.yuanzx.type.handler.demo.ListToStringTypeHandler})
   </insert>

    <select id="deptFind" resultMap="deptMap">
        select * from dept
   </select>

```

# 3 测试运行

```java
// Lombok 注解
@Data
@Accessors(chain = true)
public class Dept {
    private Integer deptNo;
    private String dname;
    private String loc;
    private Boolean flag;
    private List idList;
}

public class TestMain {
    private SqlSession session;

    @Before
    public void start() {
        try {
            InputStream inputStream = Resources.getResourceAsStream("myBatis-config.xml");
            SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
            session = factory.openSession();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

    @Test
    public void test1() {
        DeptMapper dao = session.getMapper(DeptMapper.class);
        Dept dept = new Dept();
        dept.setIdList(CollectionUtil.toList(1, 2, 3, 4, 5, 6, 7));
        dao.deptSave(dept);
        session.commit();
    }

    @Test
    public void test2() {
        DeptMapper dao = session.getMapper(DeptMapper.class);
        List<Dept> deptList = dao.deptFind();
        for (Dept dept : deptList) {
            System.out.println(dept.toString());
        }

    }

    @After
    public void end() {
        if (session != null) {
            session.close();
        }
    }
}
```


# 3 JdbcType 的选择

在 Mybatis 中已经定义好了枚举类，直接调用即可

```java
public enum JdbcType {
  ARRAY(Types.ARRAY),
  BIT(Types.BIT),
  TINYINT(Types.TINYINT),
  SMALLINT(Types.SMALLINT),
  INTEGER(Types.INTEGER),
  BIGINT(Types.BIGINT),
  FLOAT(Types.FLOAT),
  REAL(Types.REAL),
  DOUBLE(Types.DOUBLE),
  NUMERIC(Types.NUMERIC),
  DECIMAL(Types.DECIMAL),
  CHAR(Types.CHAR),
  VARCHAR(Types.VARCHAR),
  LONGVARCHAR(Types.LONGVARCHAR),
  DATE(Types.DATE),
  TIME(Types.TIME),
  TIMESTAMP(Types.TIMESTAMP),
  BINARY(Types.BINARY),
  VARBINARY(Types.VARBINARY),
  LONGVARBINARY(Types.LONGVARBINARY),
  NULL(Types.NULL),
  OTHER(Types.OTHER),
  BLOB(Types.BLOB),
  CLOB(Types.CLOB),
  BOOLEAN(Types.BOOLEAN),
  CURSOR(-10), // Oracle
  UNDEFINED(Integer.MIN_VALUE + 1000),
  NVARCHAR(Types.NVARCHAR), // JDK6
  NCHAR(Types.NCHAR), // JDK6
  NCLOB(Types.NCLOB), // JDK6
  STRUCT(Types.STRUCT),
  JAVA_OBJECT(Types.JAVA_OBJECT),
  DISTINCT(Types.DISTINCT),
  REF(Types.REF),
  DATALINK(Types.DATALINK),
  ROWID(Types.ROWID), // JDK6
  LONGNVARCHAR(Types.LONGNVARCHAR), // JDK6
  SQLXML(Types.SQLXML), // JDK6
  DATETIMEOFFSET(-155); // SQL Server 2008
}
```