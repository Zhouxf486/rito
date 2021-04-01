## hahah

### 1

### 2

```java
package com.iskeen.advert.platform.controller;


import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.google.common.collect.Maps;
import com.iskeen.advert.platform.common.annotation.Log;
import com.iskeen.advert.platform.common.constant.Constant;
import com.iskeen.advert.platform.common.dto.Result;
import com.iskeen.advert.platform.common.token.TokenThreadLocal;
import com.iskeen.advert.platform.core.domain.Creativity;
import com.iskeen.advert.platform.core.domain.User;
import com.iskeen.advert.platform.core.domain.dto.AdStatusDTO;
import com.iskeen.advert.platform.core.domain.dto.CreativeDTO;
import com.iskeen.advert.platform.core.domain.query.CreativityQuery;
import com.iskeen.advert.platform.core.domain.vo.CreativityVO;
import com.iskeen.advert.platform.core.service.CreativityService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import java.time.LocalDate;
import java.util.List;
import java.util.Map;
import java.util.regex.Pattern;
import java.util.stream.Collectors;

/**
 * 广告创意(Creativity)表控制层
 *
 * @author zhaobo
 * @since 2020-09-09 10:56:24
 */
@RestController
@RequestMapping("/api/creativity")
public class CreativityController {

    private static final String REGEX = "^\\d{19}$";

    /**
     * 服务对象
     */
    @Resource
    private CreativityService creativityService;

    /**
     * 分页查询所有数据
     *
     * @param page  分页对象
     * @param query 查询实体
     * @return 所有数据
     */
    @GetMapping
    public ResponseEntity selectAll(Page<CreativityVO> page, CreativityQuery query) {

        if (page.getCurrent() == -1) {
            Map<String, Object> params = Maps.newHashMapWithExpectedSize(2);
            if (query.getPlanId() != null) {
                params.put("plan_id", query.getPlanId());
            }
            List<Creativity> creativityList = this.creativityService.listByMap(params);
            if (query.getStatus() == -1) {
                creativityList = creativityList.stream()
                        .filter(cre -> cre.getStatus() > -99)
                        .collect(Collectors.toList());
            }
            return Result.ok(creativityList);
        }

        User user = TokenThreadLocal.get();
        query.setUserId(user.getId());
        if (StringUtils.isNotEmpty(query.getKeyword())) {
            String keyword = query.getKeyword();
            if (Pattern.matches(REGEX, keyword)) {
                query.setId(Long.parseLong(keyword));
            } else {
                query.setName(keyword);
            }
        }
        if (StringUtils.isAnyEmpty(query.getStartDate(), query.getEndDate())) {
            String now = LocalDate.now().toString();
            query.setStartDate(now);
            query.setEndDate(now);
        }
        return Result.ok(this.creativityService.query(page, query));
    }

    /**
     * 通过主键查询单条数据
     *
     * @param id 主键
     * @return 单条数据
     */
    @GetMapping("{id}")
    public ResponseEntity selectOne(@PathVariable Long id) {
        return Result.ok(this.creativityService.getCreativeInfo(id));
    }

    /**
     * 新增数据
     *
     * @param creativeDTO 实体对象
     * @return 新增结果
     */
    @PostMapping
    @Log(targetType = Constant.CREATIVITY, action = Constant.CREATE, description = "新建广告创意")
    public ResponseEntity insert(@RequestBody @Validated CreativeDTO creativeDTO) {
        User user = TokenThreadLocal.get();
        creativeDTO.setUserId(user.getId());
        return Result.ok(this.creativityService.save(creativeDTO));
    }

    /**
     * 修改数据
     *
     * @param creativeDTO 实体对象
     * @return 修改结果
     */
    @PutMapping("{id}")
    @Log(targetType = Constant.CREATIVITY, action = Constant.UPDATE, description = "修改广告计划")
    public ResponseEntity update(@RequestBody @Validated CreativeDTO creativeDTO,
                                 @PathVariable Long id) {
        User user = TokenThreadLocal.get();
        creativeDTO.setUserId(user.getId());
        return Result.ok(this.creativityService.updateById(id, creativeDTO));
    }

    @Log(targetType = Constant.CREATIVITY, action = Constant.STATUS_CHANGE, description = "广告创意状态变更")
    @PatchMapping("/status")
    public ResponseEntity updateStatus(@RequestBody @Validated AdStatusDTO statusDTO) {
        User user = TokenThreadLocal.get();
        return Result.ok(this.creativityService
                .updateStatus(user.getId(), null, null, statusDTO.getIdList(),
                        Integer.valueOf(statusDTO.getStatus())));
    }


    /**
     * 功能描述: 创意配置浮动值
     *
     * @param volatility
     * @return: org.springframework.http.ResponseEntity
     * @Author: zhouxiangfu
     * @Date: 2020/10/30 4:46 下午
     */
    @PatchMapping("/volatility")
    public ResponseEntity addVolatility(@RequestParam Long creativityId,
                                        @RequestParam Double volatility) {


        creativityService.addVolatility(creativityId, volatility);

        return Result.ok("配置成功！");
    }


    /**
     * 功能描述: 全局创意配置浮动值
     *
     * @param volatility
     * @return: org.springframework.http.ResponseEntity
     * @Author: zhouxiangfu
     * @Date: 2020/10/30 4:46 下午
     */
    @PatchMapping("/volatility/global")
    public ResponseEntity addGlobalVolatility(@RequestParam Double volatility) {


        creativityService.addGlobalVolatility(volatility);

        return Result.ok("配置成功！");
    }


    /**
     * 功能描述: 用户创意配置浮动值
     *
     * @param volatility
     * @return: org.springframework.http.ResponseEntity
     * @Author: zhouxiangfu
     * @Date: 2020/10/30 4:46 下午
     */
    @PatchMapping("/volatility/user")
    public ResponseEntity addUserVolatility(@RequestParam Double volatility, @RequestParam Integer userId) {


        creativityService.addUserVolatility(volatility,userId);

        return Result.ok("配置成功！");
    }


    /**
     * 功能描述: 创意配置自定义id
     *
     * @param customId
     * @return: org.springframework.http.ResponseEntity
     * @Author: zhouxiangfu
     * @Date: 2020/10/30 4:46 下午
     */
    @PatchMapping("/customId/{creativityId}")
    public ResponseEntity addCustomId(@PathVariable Long creativityId, @RequestParam String customId) {

        creativityService.addCustomId(creativityId, customId);
        return Result.ok("配置成功！");
    }

}
```

