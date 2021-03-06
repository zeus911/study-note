ETL 34种子系统
==================
+ 抽取
	1. 子系统1： 数据剖析系统`分析数据源结构和内容.`
		* 列特征分析
			1. 一列有多少不同值
			2. 空值个数
			3. 最大最小值
			4. 数值类型的合计,中位数,标准差等
			5. 字符串模式和长度
			6. 单次个数 大小写字符个数
			7. 词频统计
		* 依赖性特征分析
		* 连接特征分析
		* 工具 `Data cleaner`
	2. 子系统2： 增量数据捕获系统
		* 基于数据源的CDC（时间戳 系列）
		* 基于触发器的CDC
		* 基于快照的CDC（table input + merge row）
		* 基于日志的CDC
	3. 子系统3:  抽取系统
		* 基于文件
		* 基于流
+ 清洗和更正
	1. 子系统4: 数据清洗和质量处理系统
	2. 子系统5: 错误事件处理系统
	3. 子系统6: 审计纬度
	4. 子系统7: 清楚重复记录系统
	5. 子系统8: 数据一致性
+ 发布
	1. 子系统9: 缓慢变更纬度处理
		* update
		* insert(原记录记为old)
		* add_column
		* 增加一个小维度表
		* 分离历史表
		* 混合型（结合update、insert和addclumn）
	2. 子系统10: 代理价生成系统(适用于创建或第一次加载维度表)
		* 表里代理键最大值加1
		* 数据库序列
		* 自增字段
	3. 子系统11: 层次唯独创建	
	4. 子系统12: 特殊纬度生成
		* 杂项纬度
		* 小唯独
		* 收缩或上卷的纬度
		* 静态纬度
		* 用户自定义纬度
	5. 子系统13: 事实表加载
		* 事务粒度事实表
		* 周期快照事实表  
		* 累计快照事实表
	6. 子系统14: 代理键管道
	7. 子系统15: 多值维度桥接表生成纬度
	8. 子系统16: 迟到数据处理
	9. 子系统17: 纬度管理系统
	10. 子系统18: 事实表管理系统
	11. 子系统19: 聚集构建
	12. 子系统20: OLAP Cube构建系统
	13. 子系统21: 数据整合管理系统
+ 管理
	1. 子系统22: 作业调度
	2. 子系统23: 备份系统
		* 源数据
		* 转换后数据
		* 加载前数据
	3. 子系统24: 恢复和重新启动系统
	4. 子系统25: 版本控制系统
	5. 子系统26: 从开发环境到测试 产品环境迁移
	6. 子系统27: 工作流监控
	7. 子系统28: 排序系统
	8. 子系统29: 血统和依赖分析
	9. 子系统30: 问题报告系统
	10. 子系统31: 并行/管道系统
	11. 子系统32: 安全系统
	12. 子系统33: 合规报告系统
	13. 子系统34: 元数据资源库管理系统