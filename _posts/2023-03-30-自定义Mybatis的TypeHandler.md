---
title: 自定义Mybatis的TypeHandler
categories:
- java
- mybatis
tags:
- 瞎折腾
---
刚开始使用若依的时候碰到了这样的一个问题：数据库中status字段存储的是1、2、3...这样的值，对应的枚举在枚举类中，在查询的时候怎么样给前端返回对应的枚举？因为刚学习完了mybatis所以就想到用typehandler来处理对应的字段。但是这种枚举首选的解决办法貌似应该是建立枚举表去关联查询这样，随着后来对若依框架的深入使用发现其使用了**字典管理**来对这些枚举进行统一的增删改查，这些字典在系统登录之后会缓存在redis中，在查询数据时只需要将表中数据原封不动的查出来再使用其前端dict组件展示即可，不需要去做关联查询。当然也更不需要自定义TypeHandler ┭┮﹏┭┮ 我更像是学了mybatis之后的一次瞎折腾，但是还是将我第一次折腾typehandler的过程记录一下，供日后批判。
<!-- more -->

## TypeHandler

  所有的类型转换器都实现了TypeHandler这个接口，TypeHandler中有4个方法，可分为两类：其中setParameter方法及其重载方法的作用是将Java类型转换为JDBC类型；getResult方法的作用是将JDBC类型转换成Java类型。具体如下：

```java

  /**
   * 类型处理器：
   *   JAVA类型   <------>  JDBC类型
   *       JAVA类型 ----> JDBC类型    写
   *       JDBC类型 ----> JAVA类型    读
   *     SQL操作：读 写
   * 负责将Java类型转换为JDBC的类型
   *    本质上执行的就是JDBC操作中的 如下操作
   *        String sql = "SELECT id,user_name,real_name,password,age,d_id from t_user where id = ? and user_name = ?";
   *        ps = conn.prepareStatement(sql);
   *        ps.setInt(1,2);
   *        ps.setString(2,"张三");
   * @param ps
   * @param i 对应占位符的 位置
   * @param parameter 占位符对应的值
   * @param jdbcType 对应的 jdbcType 类型
   * @throws SQLException
   */
  void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;

  /**
   * 从ResultSet中获取数据时会调用此方法，会将数据由JdbcType转换为Java类型
   * ResultSet rs = ps.executeQuery();
   * rs.next;
   * rs.getString(columnName);
   * rs.getInteger(columnIndex)
   *
   * @param columnName Colunm name, when configuration <code>useColumnLabel</code> is <code>false</code>
   */
  T getResult(ResultSet rs, String columnName) throws SQLException;

  T getResult(ResultSet rs, int columnIndex) throws SQLException;

  T getResult(CallableStatement cs, int columnIndex) throws SQLException;

```

## BaseTypeHandler

   BaseTypeHandler继承了TypeReference抽象类，实现了TypeHandler接口，它本身仍然是抽象类，在它内部简单的实现了TypeHandler接口中定义的四个方法中在所有类型处理器都需要考虑的公共代码。

- 字段

```java
  /**
   * @deprecated Since 3.5.0 - See https://github.com/mybatis/mybatis-3/issues/1203. This field will remove future.
   */
  @Deprecated
  protected Configuration configuration;

  /**
   * @deprecated Since 3.5.0 - See https://github.com/mybatis/mybatis-3/issues/1203. This property will remove future.
   */
  @Deprecated
  public void setConfiguration(Configuration c) {
    this.configuration = c;
  }
```

- setParameter()方法

  该方法主要实现了在SQL语句执行时，参数转换成jdbc对应的类型。该方法处理了当参数为空的情况，非空的情况交由setNonNullParameter()方法实现，该方法是抽象方法，需要在实现类中实现。

```java

  @Override
  public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
    if (parameter == null) {
      if (jdbcType == null) {
        throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");
      }
      try {
        ps.setNull(i, jdbcType.TYPE_CODE);
      } catch (SQLException e) {
        throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . "
              + "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. "
              + "Cause: " + e, e);
      }
    } else {
      try {
        setNonNullParameter(ps, i, parameter, jdbcType);
      } catch (Exception e) {
        throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . "
              + "Try setting a different JdbcType for this parameter or a different configuration property. "
              + "Cause: " + e, e);
      }
    }
  }
```

```java
  public abstract void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;
```

- getResult()方法及重载方法

  该方法主要实现了在SQL语句执行后，结果集转换成JAVA对应的类型。该方法通过getNullableResult()方法真正实现结果集的转换，该方法是抽象方法，需要在子类中实现。当结果集为null时，直接返回了null。剩下的两个重载方法，逻辑一样，不再贴代码。

```java
@Override
  public T getResult(ResultSet rs, String columnName) throws SQLException {
    try {
      return getNullableResult(rs, columnName);
    } catch (Exception e) {
      throw new ResultMapException("Error attempting to get column '" + columnName + "' from result set.  Cause: " + e, e);
    }
  }

  @Override
  public T getResult(ResultSet rs, int columnIndex) throws SQLException {
    try {
      return getNullableResult(rs, columnIndex);
    } catch (Exception e) {
      throw new ResultMapException("Error attempting to get column #" + columnIndex + " from result set.  Cause: " + e, e);
    }
  }

  @Override
  public T getResult(CallableStatement cs, int columnIndex) throws SQLException {
    try {
      return getNullableResult(cs, columnIndex);
    } catch (Exception e) {
      throw new ResultMapException("Error attempting to get column #" + columnIndex + " from callable statement.  Cause: " + e, e);
    }
  }
```

```java
 /**
   * @param columnName Colunm name, when configuration <code>useColumnLabel</code> is <code>false</code>
   */
  public abstract T getNullableResult(ResultSet rs, String columnName) throws SQLException;

  public abstract T getNullableResult(ResultSet rs, int columnIndex) throws SQLException;

  public abstract T getNullableResult(CallableStatement cs, int columnIndex) throws SQLException;
```

## 简单实现自定义类型解析器

  这个解析器会将字段名称符合的字段进行值的转换。

```java
/**
 * @author: LiuBowen
 * @date: 2023/03/29
 * @description: 自定义计划类型状态处理器，用于将tinyint类型的类型转换成{type:"xxx",label:"xxx"}
 */
public class StatusTypeHandler extends BaseTypeHandler<Map> {


    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Map map, JdbcType jdbcType) throws SQLException {
        //插入

        preparedStatement.setInt(i, PlanStatusEnum.getValueByLabel(MapUtils.getString(map,"label")));
    }

    @Override
    public Map getNullableResult(ResultSet resultSet, String s) throws SQLException {
        //查询
        if (s.equals(PlanColoumNameEnum.PLAN_TYPE.getName())){
            int anInt = resultSet.getInt(s);
            return PlanTypeEnum.getDescByValue(anInt);
        }
        if (s.equals(PlanColoumNameEnum.STATUS.getName())){
            int anInt = resultSet.getInt(s);
            return PlanStatusEnum.getDescByValue(anInt);
        }
        return new HashMap();
    }

    @Override
    public Map getNullableResult(ResultSet resultSet, int i) throws SQLException {
        //查询
        int anInt = resultSet.getInt(i);
        return PlanStatusEnum.getDescByValue(anInt);
    }

    @Override
    public Map getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        //查询
        int anInt = callableStatement.getInt(i);
        return PlanStatusEnum.getDescByValue(anInt);
    }
}
```
