## 注解说明

```java
@Data
@Entity(name = "legalperson")
public class LegalPerson {
    @Id
    @GeneratedValue
    private Integer id;
//    private Integer uid;//用户外键
    private String name;
    private String code;
    private Integer status;

    @Transient // 创建表时，忽略此字段
    private Integer statusName;
    
    @ManyToOne
    @JoinColumn(name = "uid")
    private UserInfo userInfo;
}

@Data
@Entity(name = "userinfo")
public class UserInfo {

    @Id
    @GeneratedValue
    private Integer id;
    private String name;
    private Integer age;
    private String sex;

    public UserInfo() {
    }

    public UserInfo(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}

```

