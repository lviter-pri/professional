## 一、MapStruct简介

### 1.1 MapStruct与各主流框架对比

MapStruct是一款高性能的对象映射框架，采用JSR269注解处理器，支持可配置化、扩展性强。

MapStruct的高性能得益于预生成方式，采用原生Getter/Setter方法进行赋值，不会产生额外开销损耗

| **工具**             | **原理**                                                    | **性能特征**                |
| -------------------- | ----------------------------------------------------------- | --------------------------- |
| **MapStruct**        | **编译期生成代码**：生成与手动 `set/get` 等效的 Java 代码。 | **最高**（等同于原生编码）  |
| **Spring BeanUtils** | **运行时反射**：在运行时动态扫描并赋值。                    | **中等**                    |
| **Hutool BeanUtil**  | **运行时反射/缓存**：对反射做了封装与缓存优化。             | **中等偏高**（优于 Spring） |

### 1.2 性能测算估算 (耗时对比)

以下数据基于常见开发环境的基准测算值

| **循环次数** | **MapStruct (ms)** | **Hutool BeanUtil (ms)** | **Spring BeanUtils (ms)** |
| ------------ | ------------------ | ------------------------ | ------------------------- |
| **1 次**     | < 1                | ~ 5 - 10                 | ~ 10 - 20                 |
| **1 万次**   | ~ 5                | ~ 80 - 150               | ~ 200 - 400               |
| **100 万次** | ~ 50 - 100         | ~ 8,000 - 12,000         | ~ 15,000 - 25,000         |
| **500 万次** | ~ 250 - 500        | ~ 40,000 - 60,000        | ~ 80,000 - 120,000        |

### 1.3 优缺点分析

#### 优点：为何它是现代架构的首选

1. **极致的执行性能**：
   - **原理**：它不使用反射。生成的 `.class` 文件中全是纯粹的 Java 方法调用。
   - **结果**：性能几乎等同于手写代码，比 `BeanUtils` 或 `ModelMapper` 快 10-100 倍。
2. **编译期类型检查**：
   - **提前纠错**：如果你把 `String` 映射到 `Long` 且没有提供转换逻辑，**编译会报错**。你不需要等到程序运行到那一时刻才发现空指针或类型转换异常。
3. **零运行时依赖**：
   - 生成的代码是原生 Java，运行时不需要额外的库支撑，对包体积和启动速度极其友好。
4. **调试透明化**：
   - 你可以直接打开 `target/generated-sources` 目录查看它生成的 `.java` 文件。逻辑不通时，打断点进去看一眼便知，告别“反射黑盒”。
5. **强大的多源映射能力**：
   - 它能轻松将 3 个实体（Bill, Order, Adjustment）聚合成 1 个 VO。



#### 缺点：需要面对的挑战

1. **引入成本与学习曲线**：
   - 需要配置 Maven/Gradle 的 `annotationProcessor`。
   - 对于复杂的映射（如嵌套对象、循环依赖、特殊格式转换），需要学习其特定的注解（如 `@Mapping` 的表达式、`qualifiedByName` 等）。
2. **代码膨胀与编译耗时**：
   - 由于是生成代码，Mapper 接口越多，生成的实现类就越多，会导致项目全量编译时间略微增加。
3. **对重构的依赖度高**：
   - 如果你修改了实体的字段名，必须重新编译才能发现 Mapper 中的报错。虽然 IDE 插件（如 MapStruct Support）能缓解这一点，但它仍然比动态映射工具更依赖静态检查。
4. **不够灵活**：
   - 如果你需要基于某些运行时动态配置来决定映射关系（例如根据权限动态隐藏字段），MapStruct 的静态生成属性会显得比较死板，通常需要配合 `default` 方法手动补丁。

| **方案**                    | **适用场景**                                 | **结论**                             |
| --------------------------- | -------------------------------------------- | ------------------------------------ |
| **手写 Set/Get**            | 字段极少（<3个），代码量极小。               | 稳，但太费手。                       |
| **BeanUtils / ModelMapper** | 快速原型开发，性能要求极低。                 | **架构师禁止使用**（反射坑多且慢）。 |
| **MapStruct**               | **企业级微服务、高性能网关、复杂业务聚合。** | **首选方案。**                       |

## 二、MapStruct接入示例

2.1 pom引入依赖

```xml
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>1.6.3</version>
            </dependency>
            <!--mapstruct编译-->
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.6.3</version>
            </dependency>
```

2.2 定义mapper映射

```java
/**
 * 车辆信息
 *
 * @author Lv.
 * @date 2026/4/1 16:01
 */
@Mapper(nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS)
public interface JyhCarInfoSyncConvert {

    JyhCarInfoSyncConvert INSTANCE = Mappers.getMapper(JyhCarInfoSyncConvert.class);

    /**
     * 构建车辆信息
     *
     * @param vehicleInfoDTO 车辆信息
     * @return 数据同步
     */
    @Mapping(source = "vehicleInfoDTO", target = "vehicleBasicDomain")
    @Mapping(target = "vehicleAuthDomain", expression = "java(toAuthDomain(vehicleInfoDTO, guaPlateNumber, guaImageUrl, guaImageSideUrl))")
    JyhCarInfoSyncReq toSyncReq(VehicleInfoDTO vehicleInfoDTO, String guaPlateNumber, String guaImageUrl, String guaImageSideUrl);


    @Mapping(target = "licensePlateNumber", source = "vehicle.plateNumber")
    @Mapping(target = "licensePlateColor", expression = "java(calculatedPlateColor(vehicleInfoDTO))")
    VehicleBasicDomain toBasicDomain(VehicleInfoDTO vehicleInfoDTO);

    @Mapping(target = "vehicleType", expression = "java(calculatedVehicleType(vehicleInfoDTO.getVehicle()))")
    @Mapping(target = "vehicleLength", source = "vehicleInfoDTO.vehicle.carLength", qualifiedByName = "stringToBigDecimal")
    @Mapping(target = "vehicleEnergyType", expression = "java(calculatedEnergyType(vehicleInfoDTO.getCertDrivingLicense()))")
    @Mapping(target = "vehicleIdentificationNumber", source = "vehicleInfoDTO.vehicle.vin")
    @Mapping(target = "vehicleLicenseFrontImg", source = "vehicleInfoDTO.certDrivingLicense.imageUrl")
    @Mapping(target = "vehicleLicenseBackImg", source = "vehicleInfoDTO.certDrivingLicense.sideImageUrl")
    @Mapping(target = "vehicleLicenseClipImg", source = "vehicleInfoDTO.certDrivingLicense.annualCheckImageUrl")
    @Mapping(target = "vehicleLicenseRegisterTime", source = "vehicleInfoDTO.certDrivingLicense.registerDate", qualifiedByName = "aimTimeConvert")
    @Mapping(target = "vehicleLicenseExpireTime", source = "vehicleInfoDTO.certDrivingLicense.validDate", qualifiedByName = "aimTimeConvert2")
    @Mapping(target = "vehicleLicenseIssuingAuthority", source = "vehicleInfoDTO.certDrivingLicense.issuingAuthority")
    @Mapping(target = "vehicleLicenseIssuingTime", source = "vehicleInfoDTO.certDrivingLicense.issueDate", qualifiedByName = "aimTimeConvert")
    @Mapping(target = "roadTransportLicense", source = "vehicleInfoDTO.certRoadTransportLicense.cardNumber")
    @Mapping(target = "roadTransportLicenseImg", source = "vehicleInfoDTO.certRoadTransportLicense.imageUrl")
    @Mapping(target = "roadTransportLicenseAnnualInspectionImg", source = "vehicleInfoDTO.certRoadTransportLicense.imageUrl")
    @Mapping(target = "roadTransportLicenseExpireTime", source = "vehicleInfoDTO.certRoadTransportLicense.endValidDate", qualifiedByName = "aimTimeConvert")
    @Mapping(target = "roadTransportBusinessLicense", source = "vehicleInfoDTO.certRoadTransportLicense.roadOperationLicenseNumber")
    @Mapping(target = "trailerLicenseNumber", source = "guaPlateNumber")
    @Mapping(target = "trailerLicenseImg", source = "guaImageUrl")
    @Mapping(target = "trailerLicenseBackImg", source = "guaImageSideUrl")

    @Mapping(target = "approvedLoad", expression = "java(calculatedApprovedLoad(vehicleInfoDTO.getCertDrivingLicense()))")
    @Mapping(target = "curbWeight", expression = "java(calculatedCurbWeight(vehicleInfoDTO.getCertDrivingLicense()))")
    @Mapping(target = "totalMass", expression = "java(sumMass(calculatedApprovedLoad(vehicleInfoDTO.getCertDrivingLicense()), calculatedCurbWeight(vehicleInfoDTO.getCertDrivingLicense())))")
    @Mapping(target = "overallDimensions", source = "vehicleInfoDTO.certDrivingLicense.externalSize")
    @Mapping(target = "useNature", expression = "java(calculatedUseNature(vehicleInfoDTO.getCertDrivingLicense()))")
    @Mapping(target = "vehicleOwner", source = "vehicleInfoDTO.vehicle.vehicleOwner")
    VehicleAuthDomain toAuthDomain(VehicleInfoDTO vehicleInfoDTO, String guaPlateNumber, String guaImageUrl, String guaImageSideUrl);

    @Named("stringToBigDecimal")
    default BigDecimal stringToBigDecimal(String source) {
        if (source == null || source.trim().isEmpty()) {
            return null;
        }
        try {
            // 过滤掉可能存在的非数字字符（如“米”），确保转换成功
            String cleaned = source.replaceAll("[^0-9.]", "");
            return new BigDecimal(cleaned);
        } catch (NumberFormatException e) {
            return null;
        }
    }

    @Named("aimTimeConvert2")
    default String aimTimeConvert2(String time) {
        return uniformDateFmt(time, DatePattern.NORM_MONTH_PATTERN);
    }

    @Named("aimTimeConvert")
    default String aimTimeConvert(String time) {
        return uniformDateFmt(time, DatePattern.NORM_DATE_PATTERN);
    }

    default BigDecimal sumMass(BigDecimal a, BigDecimal b) {
        if (a == null && b == null) {
            return null;
        }
        return (a == null ? BigDecimal.ZERO : a).add(b == null ? BigDecimal.ZERO : b);
    }

    /**
     * 整备
     *
     * @param certDrivingLicense 证件
     * @return 整备
     */
    default BigDecimal calculatedCurbWeight(CertDrivingLicense certDrivingLicense) {
        return NdcTools.weight2TonOrNull(certDrivingLicense.getCurbWeight());
    }

    /**
     * 总质量
     *
     * @param certDrivingLicense 证件
     * @return 总质量
     */
    default BigDecimal calculatedTotalMass(CertDrivingLicense certDrivingLicense) {
        return NdcTools.weight2TonOrNull(certDrivingLicense.getTotalMass());
    }

    /**
     * 核载
     *
     * @param certDrivingLicense 证件
     * @return 核载
     */
    default BigDecimal calculatedApprovedLoad(CertDrivingLicense certDrivingLicense) {
        //优先核载，其次牵引总质量
        BigDecimal weight = NdcTools.weight2TonOrNull(certDrivingLicense.getLoadQuality());
        if (Objects.isNull(weight)) {
            return NdcTools.weight2TonOrNull(certDrivingLicense.getTotalQuasiMass());
        }
        return weight;
    }

    default String calculatedUseNature(CertDrivingLicense certDrivingLicense) {
        if (Objects.isNull(certDrivingLicense)) {
            return null;
        }
        String natureOfUse = null;
        if (StringUtils.isNotBlank(certDrivingLicense.getNatureOfUse())) {
            natureOfUse = Arrays.stream(NatureOfUseEnum.values())
                    .filter(natureOfUseEnum -> natureOfUseEnum.getCode().equals(certDrivingLicense.getNatureOfUse()))
                    .findFirst()
                    .map(NatureOfUseEnum::getDesc)
                    .orElse(null);
        }
        return natureOfUse;
    }

    /**
     * 自定义处理规则:
     *
     * @param vehicleInfo 车辆信息
     * @return 车牌颜色
     */
    default String calculatedPlateColor(VehicleInfoDTO vehicleInfo) {

        if (Objects.isNull(vehicleInfo)) {
            return null;
        }
        Vehicle vehicle = vehicleInfo.getVehicle();
        if (Objects.isNull(vehicle)) {
            return null;
        }
        // 车牌颜色 枚举：黄色；蓝色；绿色；黄绿色
        String plateColorCode = PlateColorEnum.parse(vehicle.getPlateColor()).getCode();

        String plateColor = null;
        if (StringUtils.isNotBlank(plateColorCode)) {
            // 将 黄牌 替换为 黄色 以此类推其他颜色
            if (PlateColorEnum.YELLOW.getCode().equals(plateColorCode)) {
                plateColor = VehicleBasicDomain.LicensePlateColorEnum.YELLOW.getCode();
            } else if (PlateColorEnum.GREEN.getCode().equals(plateColorCode)) {
                plateColor = VehicleBasicDomain.LicensePlateColorEnum.GREEN.getCode();
            } else if (PlateColorEnum.BLUE.getCode().equals(plateColorCode)) {
                plateColor = VehicleBasicDomain.LicensePlateColorEnum.BLUE.getCode();
            } else if (PlateColorEnum.YELLOW_GREEN.getCode().equals(plateColorCode)) {
                plateColor = VehicleBasicDomain.LicensePlateColorEnum.GREEN.getCode();
            }
        }
        return plateColor;
    }

    /**
     * 自定义处理规则:
     *
     * @param vehicle 车辆信息
     * @return 车辆类型
     */
    default String calculatedVehicleType(Vehicle vehicle) {
        if (Objects.isNull(vehicle)) {
            return null;
        }
        return AffiliationHelper.getJyhVehicleType(vehicle.getVehicleName());
    }

    /**
     * 能源类型映射
     *
     * @param certDrivingLicense 证件信息
     * @return 能源类型
     */
    default String calculatedEnergyType(CertDrivingLicense certDrivingLicense) {
        if (Objects.isNull(certDrivingLicense)) {
            return null;
        }

        FuelTypeEnum fuelTypeEnum = FuelTypeEnum.parse(certDrivingLicense.getFuelType());
        if (Objects.isNull(fuelTypeEnum)) {
            return null;
        }
        switch (fuelTypeEnum) {
            case GASOLINE:
                return VehicleAuthDomain.VehicleEnergyTypeEnum.A.getCode();
            case DIESEL_OIL:
                return VehicleAuthDomain.VehicleEnergyTypeEnum.B.getCode();
            case ELECTRICAL:
                return VehicleAuthDomain.VehicleEnergyTypeEnum.C.getCode();
            case NATURAL_GAS:
                return VehicleAuthDomain.VehicleEnergyTypeEnum.F.getCode();
            case OIL_ELECTRICAL:
                return VehicleAuthDomain.VehicleEnergyTypeEnum.D.getCode();
            case GAS:
                return VehicleAuthDomain.VehicleEnergyTypeEnum.E.getCode();
            default:
                return VehicleAuthDomain.VehicleEnergyTypeEnum.Z.getCode();
        }
    }
    
    public String uniformDateFmt(String date, String format) {
		if (StrUtil.isBlank(date)) {
			return null;
		}
		try {
			return DateUtil.parse(date).toString(format);
		} catch (Exception e) {
			log.error("日期({})格式错误, 无法转换为{}格式", date, format);
			return date;
		}
	}
}

```

2.3 显式调用转换方法

```java
AffiliationSystemConfigPO po = AffiliationSystemConfigConvert.INSTANCE.convert(affiliationSystemConfigDTO);
```



