---
title: MapStruct实体转换
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(116).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(116).jpg
tags:
  - 大数据
  - 代码整洁
category:
  - - 编程
    - JAVA
  - - 编程
    - 代码整洁
date: 2022-02-28 11:19:58
---

在项目中，我们经常会用到`VO`、`DTO`、`PO`等等`对象之间的相互转换`。

可用的工具有`BeanUtils`、`MapStruct`等。`BeanUtils`是傻瓜式的很死板。

今天记录下`MapStruct`的`简单用法`。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 官网

https://mapstruct.org/

> 可以给idea装个插件`MapStruct Support`,这样会有一些提示

## 2 POM

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.4.2.Final</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.4.2.Final</version>
</dependency>
```

## 3 单个源

### 3.1 定义实体

> 定义了好几个实体，我就截个图吧，方便对比，代码可以去`github`拉

https://github.com/xiaoma55/CODE_MapStruct

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/MapStruct实体转换/1.png)

### 3.2 定义接口

```
@Mapper(componentModel = "spring")
public interface ICarMapper {

    /* 可以用下面这个实例化，然后调用的时候就是 ICarMapper.INSTANCE.******方法，
       ICarMapper INSTANCE = Mappers.getMapper(ICarMapper.class);

       但是这样有点违背了spring的编程风格，我们可以使用另一种方式
       我们一般在接口加 @Mapper(componentModel = "spring")
       这样就可以 @Autowired去注入这个接口了，本篇的例子都是使用这种方式
    */


    /** carDTO2CarVO */
    @Mapping(source = "totalPrice", target = "totalPrice", numberFormat = "#.00" ,defaultValue = "￥0.00")
    @Mapping(source = "publishDate", target = "publishDate", dateFormat = "yyyy-MM-dd HH:mm:ss")
    @Mapping(target = "color", ignore = true)
    @Mapping(source = "brand", target = "brandName")
    @Mapping(source = "driverDTO", target = "driverVO")
    CarVO carDTO2CarVO(CarDTO carDTO);

    /** 批量转换 */
    List<CarVO> carDTOs2carVOs(List<CarDTO> carDTO);

    /** driverDTO2DriverVO */
    @Mapping(source = "id", target = "driverId")
    @Mapping(source = "name", target = "fullName")
    DriverVO driverDTO2DriverVO(DriverDTO driverDTO);

    /** 表示让mapstruct在调用完自动转换的方法之后，会来自动调用本方法 */
    @AfterMapping
    default void dto2voAfter(CarDTO carDTO, @MappingTarget CarVO carVO) {
        // @MappingTarget : 表示传来的carVO对象是已经赋值过的
        List<PartDTO> partDTOS = carDTO.getPartDTOS();
        boolean hasPart = partDTOS != null && !partDTOS.isEmpty();
        carVO.setHasPart(hasPart);
    }

    /** 配置忽略mapstruct的默认映射行为，只映射那些配置了@Mapping的属性 */
    @BeanMapping(ignoreByDefault = true)
    @Mapping(source = "id", target = "id")
    @Mapping(source = "brand", target = "brandName")
    VehicleVO carDTO2vehicleVO(CarDTO carDTO);


    /** 会继承全部配置，包括@BeanMapping和@Mapping */
    @InheritConfiguration(name = "carDTO2vehicleVO")
    void updateVehicleVO(CarDTO carDTO, @MappingTarget VehicleVO vehicleVO);

    /**
     * 测试@InheritInverseConfiguration反向继承 :反转就是你原来写了dto->vo,加了这个反转就是实现了vo->dto，它会继承@Mapping的配置
     * 只会继承@Mapping,故需要再单独写一次 @BeanMapping
     * name:指定使用哪一个方法的配置,写方法的名字
     */
    @BeanMapping(ignoreByDefault = true)
    @InheritInverseConfiguration(name = "carDTO2vehicleVO")
    CarDTO vehicleVO2CarDTO(VehicleVO vehicleVO);
}
```

### 3.3 测试

```
@SpringBootTest
public class MapStructTest {
    @Autowired
    private ICarMapper carMapper;

    /** 测试 @InheritInverseConfiguration反向继承 */
    @Test
    public void test5() {
        VehicleVO vehicleVO = new VehicleVO()
                .setId(9999L)
                .setBrandName("别克")
                .setPrice(66554322d);
        CarDTO carDTO = carMapper.vehicleVO2CarDTO(vehicleVO);
        System.out.println(carDTO);
    }


    /** 测试 @InheritConfiguration继承配置 */
    @Test
    public void test4() {
        CarDTO carDTO = buildCarDTO();
        VehicleVO vehicleVO = carMapper.carDTO2vehicleVO(carDTO).setPrice(888d);
        CarDTO carDTO2 = new CarDTO()
                .setId(330L)
                .setPrice(789d)
                .setBrand("迈巴赫");
        // 通过carDTO2的属性值来更新已存在的vehicleVO对象
        carMapper.updateVehicleVO(carDTO2, vehicleVO);
        System.out.println(vehicleVO);
    }

    /** 测试 @BeanMapping */
    @Test
    public void test3() {
        CarDTO carDTO = buildCarDTO();
        VehicleVO vehicleVO = carMapper.carDTO2vehicleVO(carDTO);
        System.out.println(vehicleVO);
    }

    /** 测试 批量转换 */
    @Test
    public void test2() {
        CarDTO carDTO1 = buildCarDTO();
        CarDTO carDTO2 = new CarDTO().setId(1L);
        List<CarDTO> carDTOList = Arrays.asList(carDTO1, carDTO2);
        List<CarVO> carVOList = carMapper.carDTOs2carVOs(carDTOList);
        System.out.println(carVOList);
    }

    /** 测试 carDTO2CarVO */
    @Test
    public void test1() {
        CarDTO carDTO = buildCarDTO();
        CarVO carVO = carMapper.carDTO2CarVO(carDTO);
        System.out.println(carDTO);
        System.out.println(carVO);
    }

    /** 模拟业务构造出的CarDTO对象 */
    private CarDTO buildCarDTO() {
        // 驾驶员
        DriverDTO driverDTO = new DriverDTO()
                .setId(88L)
                .setName("小明");
        // 零件
        PartDTO partDTO1 = new PartDTO()
                .setPartId(1L)
                .setPartName("多功能方向盘");
        PartDTO partDTO2 = new PartDTO()
                .setPartId(2L)
                .setPartName("智能车门");
        // 汽车
        CarDTO carDTO = new CarDTO()
                .setId(330L)
                .setVin("vin123456789")
                .setPrice(123789.126d)
                .setTotalPrice(143789.126d)
                .setPublishDate(new Date())
                .setColor("白色")
                .setBrand("大众")
                .setDriverDTO(driverDTO)
                .setPartDTOS(Arrays.asList(partDTO1, partDTO2));
        return carDTO;
    }
}
```

## 4 多个源

### 4.1 定义实体

> 定义了好几个实体，我就截个图吧，方便对比，代码可以去`github`拉

https://github.com/xiaoma55/CODE_MapStruct

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/MapStruct实体转换/2.png)

### 4.2 定义接口

```
@Mapper(componentModel = "spring")
public interface IDeliveryAddressMapper {
    @Mapping(source = "customer.firstName", target = "forename")
    @Mapping(source = "customer.lastName", target = "surname")
    @Mapping(source = "address.street", target = "street")
    @Mapping(source = "address.postalcode", target = "postalcode")
    @Mapping(source = "address.county", target = "county")
    DeliveryAddress from(Customer customer, Address address);
}
```

### 4.3 测试

```
@SpringBootTest
public class MultiSourceTest {
    @Autowired
    private IDeliveryAddressMapper deliveryAddressMapper;

    /** 测试 @InheritInverseConfiguration反向继承 */
    @Test
    public void test1() {
        Customer customer = new Customer()
                .setFirstName("张")
                .setLastName("三");
        Address homeAddress = new Address()
                .setStreet("科技二路")
                .setCounty("雁塔区")
                .setPostalcode("89123");
        DeliveryAddress deliveryAddress = deliveryAddressMapper.from(customer, homeAddress);
        System.out.println(deliveryAddress);
    }
}
```


## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)