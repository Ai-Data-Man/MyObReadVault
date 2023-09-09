---
{"title":"mybatis plus合并两个wrapper","url":"https://blog.csdn.net/weixin_35928208/article/details/118805946","clipped_at":"2023-02-07 14:10:41","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/mybatis-plus合并两个wrapper_1675750241/","dgPassFrontmatter":true}
---

# mybatis plus合并两个wrapper

![](/img/user/阅读库藏/assets/1675750241-426136e8363e68fc321f3d3551b0d579.png)

[廖泽松](https://blog.csdn.net/weixin_35928208 "廖泽松") ![](/img/user/阅读库藏/assets/1675750241-829b50e1a754811a0f05afff88b5db50.png) 于 2021-07-16 16:33:29 发布 ![](/img/user/阅读库藏/assets/1675750241-12234b4519a7e1441526c49ab7fc9a0d.png) 1945 ![](/img/user/阅读库藏/assets/1675750241-169ac251df55845562af7f2f9151a130.png) 收藏

分类专栏： [java](https://blog.csdn.net/weixin_35928208/category_10887264.html) [mybatis plus](https://blog.csdn.net/weixin_35928208/category_11212996.html)

版权

 [![](/img/user/阅读库藏/assets/1675750241-b02eac94e08149b1c128d95ad2973db7.png) java 同时被 2 个专栏收录![](/img/user/阅读库藏/assets/1675750241-a5211edd42049e7a393169e4ddd08bf8.png)](https://blog.csdn.net/weixin_35928208/category_10887264.html "java")

9 篇文章 1 订阅

订阅专栏

 [![](/img/user/阅读库藏/assets/1675750241-317cd066951234fd1b92441893ca0b20.png) mybatis plus](https://blog.csdn.net/weixin_35928208/category_11212996.html "mybatis plus")

1 篇文章 0 订阅

订阅专栏

```java
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.core.toolkit.Constants;
import com.guotie.mdc.base.po.datumdefect.DatumDefectPO;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

import java.util.List;

/**
 * @author liaoz
 */
@Mapper
public interface DatumDefectDao extends BaseMapper<DatumDefectPO> {
    /**
     * 查询未生成基准报警的基准缺陷
     *
     * @param frameTable 关联表 可能是 data_alarm_frame_w data_alarm_frame_g data_alarm_frame_s
     */
    @Select("SELECT d.* FROM e01d d LEFT JOIN ( " +
            "SELECT f.datum_defect_id FROM ${frameTable} f LEFT JOIN ${alarmTable} a ON f.alarm_id = a.id ${ew.customSqlSegment} " +
            ") as fa ON fa.datum_defect_id = d.f01 ${sql}")
    List<DatumDefectPO> selectUnMark(@Param("frameTable") String frameTable, @Param("alarmTable") String alarmTable, @Param(Constants.WRAPPER) QueryWrapper<?> dataWrapper, @Param("sql") String sql);
}

```

```java
        QueryWrapper<DatumDefectPO> wrapper = new QueryWrapper<>();
        rangeFrameWrapper(wrapper, query);
        rangeAlarmWrapper(wrapper, query);

        QueryWrapper<DatumDefectPO> dWrapper = new QueryWrapper<>();
        rangeUnMarkWrapper(dWrapper, query);
        dWrapper.isNull(FA_HEAD + "datum_defect_id");

        //warn: 此处骚操作   作用为将两个wrapper合并
        String dSql = StrUtil.replace(dWrapper.getCustomSqlSegment(), "ew.paramNameValuePairs.", "ew.paramNameValuePairs._");
        Map<String, Object> map = wrapper.getParamNameValuePairs();
        dWrapper.getParamNameValuePairs().forEach((key, value) -> map.put("_" + key, value));

        String domainStr = "w";
        List<DatumDefectPO> pos = datumDefectDao.selectUnMark("data_alarm_frame_" + domainStr, "data_alarm_" + domainStr, wrapper, dSql);
```

  

  

显示推荐内容