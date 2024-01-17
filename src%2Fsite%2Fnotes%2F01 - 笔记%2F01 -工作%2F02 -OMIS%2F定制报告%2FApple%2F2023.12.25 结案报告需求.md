---
{"dg-publish":true,"permalink":"/01 - 笔记/01 -工作/02 -OMIS/定制报告/Apple/2023.12.25 结案报告需求/","tags":["可乐","PMO"]}
---


## 1\. 输入文件说明

> [!note] 说明
> 1. 输入文件由多个块组成（红框区域），一个块作为一组处理
> 2. 红框内包含Ad Formats、Creative Lengths、TAs、Citys、Medias等内容
> - 蓝框区域（B4:D4），为一条子任务
> - B列单元格为`1+`的，为Ad Format+Creative Length+TA+City组合
> - B列单元格为`1+ reach %`的，为Ad Format+Creative Length+TA+City+Media组合（每个块中TA+City是相同的）
> 3. 表头
> - Ad Formats & Creative Lengths
> 	由多组Ad Format + Creative Length组成，用 & 分割
> 	每组为一个Ad Format，或1个Ad Format+1个或多个Creative Length，多个Create Length用&分割
>  - TAs & Citys
> 	不固定，需要和字典表匹配，得到对应的TA和城市列表
>  - Medias
> 	每行为一个Media，该行的子任务对应的数据范围要过滤Media
> **注意**：by media部分，注意绿框区域的TA+City

![输入示例](http://static.visualmaster.com.cn/20231227202647-421251c09af4a60e9ea880580d986ef0-041bd5.png)

## 2\. 处理过程

一、加载输入文件的子任务组
1. 根据B列定位块（`1+`、`1+reach %`）
2. 拆分Ad Formats & Creative Lengths、TAs & Citys、Medias
3. 根据Ad Format、Creative Length、Media等筛选排期中的活动、点位
	1. 表头中的Ad Format可能会有别名的情况（如Masthead写为MH），通过`辅助信息表`中的别名映射
	2. 有不参与计算的点位，通过`辅助信息表`中D-H列，Ad Format+Device Targeting一起判断，Ad Format+Media Vendor一起判断
	3. 活动根据`辅助信息表`中S-U列的活动名称、活动ID筛选
4. 汇总为M+/数据中心的子任务信息

二、PM M+ & OTT数据中心
1. 筛选有Ad活动的子任务，作为M+任务输入，筛选有Tv活动的子任务，作为OTT数据中心的输入
2. 根据阈值（暂定每个任务30条子任务，提任务时可设置）拆分子任务为多组，每一组为一个M+/数据中心大任务
3. 每个任务汇总所有子任务的数据范围
4. 每个任务中的活动，固定添加所有活动所有点位的数据范围，以及所有活动点位对应的一条子任务
5. 提交任务
6. 任务结果文件处理
	1. M+输出中会将逗号转为下划线，需要转回来
	2. OTT数据中心逗号原样保留，导致csv错列，根据列数处理，生成新的csv
三、Mixreach汇总表
1. 根据M+、数据中心下载的数据填充
2. 公式部分保留原样
3. C-E列人口网民数从“人口网民数”工作表匹配
4. H-O列`imp`、`n+uv`从rawdata获取，其中all people取Stable行，具体TA取对应TA行
5. A列IGRP编号从1开始
四、跑Mixreach任务
1. 根据所有子任务的TA & tier（一般是5个）分别处理
2. 每个TA & tier对应Mixreach系统中的一个项目
3. 每个TA & tier，根据PM、OTT、TV的数据，生成curve文件，每个端口一个文件
	1. PM curve取汇总表==V、W、X==列
	2. OTT curve取==Y、Z、AA==列
	3. TV curve取==AB、AC、AD==列
4. curve文件
	1. 筛选对应列有值的行
	2. GRP取汇总表序号
	3. 曝光填0
	4. 地域根据TA & tier，national填==中国大陆==，tier填==CombineRegion==
	5. **若该端口数据都是空行，则至少放一行GRP为0，Reach也为0的数据**
5. 每个TA & tier对应到一个项目中，提一个PMOT的任务
五、从结果文件取数回填到汇总表
1. 汇总表按行取数，通过A列IGRP编号查找对应数据
2. 只有一端有数的，不处理
3. PMO mixreach（AE-AG列）
	1. PM、OTT都有数，取PM端为IGRP编号，OTT端为IGRP编号
	2. PM有数且超百，OTT没有数，取PM端为IGRP编号，OTT端为0
	3. PM没有数，OTT有数且超百，取PM端为0，OTT端为IGRP编号
	4. TV端IGRP编号为0
4. PMOT mixreach（AH-AJ列）
	1. TV端（AB列）为空，不处理
	2. TV端有数，根据PMO取数规则，以及TV端为IGRP编号
5. **==单端超百情况==**：当PM 1+Reach超过100%，OTT为空，则认为OTT是有数的，取输出文件中PM的GRP编号和OTT的GRP为0的行；反之，取PM的GRP为0和OTT的GRP编号的行


**回填信息对照表**

| IGRP | V<br>PM端-网民base | Y<br>OTT端-网民base | AB<br>TV端 | AE-AG<br>PMO mixreach | AH-AJ<br>PMOT mixreach | 说明 |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 1 |  |  |  |  |  | 全为空 |
| 2 |  |  | 10% |  |  | TV单端 |
| 3 | 10% |  |  |  |  | 单端不超百 |
| 4 | 10% |  | 10% |  | 4、0、4 | 单端不超百 |
| 5 | 102% |  |  | 5、0、0 |  | 单端超百 |
| 6 | 102% |  | 10% | 6、0、0 | 6、0、6 | 单端超百 |
| 7 | 10% | 10% |  | 7、7、0 |  | 两端有数 |
| 8 | 10% | 10% | 10% | 8、8、0 | 8、8、8 | 三端有数 |
| 9 | 102% | 10% |  | 9、9、0 |  | 两端有数有超百 |

六、汇总表回填交付报告
1. 非TV部分，取PMO 1+Reach% - 3+Reach%
	1. 如果PMO mixreach为空，取单端数据（先取P-R列，若为空则取S-U列，若S-U为空则填0），填入1+Reach%表
	2. Reach表用Reach%的数据乘以population（汇总表C列）
2. TV部分，取PMOT 1+Reach%
	1. 如果PMOT 1+Reach%为空，取PMO，若PMO为空，依次取P-R列、S-U列、0
	2. TV部分的1+Reach为1+Reach% 乘以population（汇总表C列）


## 3\. 任务参数

1. 数据范围中的起止日期，根据输入文件中的活动日期
2. 相同活动、点位的，提一条数据范围
3. 

==注意==：分组后的子任务名称中，有逗号的，M+输出数据中会将逗号转为下划线，OTT数据中心逗号原样显示，会导致csv文件错列。所以跑完的数据需要做一次处理，不然后面用数据的时候无法匹配。

## 4\. 自动化

### 4.1 提交任务

1. 输入文件：
	1. 任务文件：
		- reach%：需要跑数的TA & tier、AdFormat、Media等信息
		- 辅助信息表：AdFormat别名、不参与计算的参数、TA对照、TA城市对照、活动时间
	1. TV数据：TA & tier对应的GRP、Reach数据


### 4.2 任务执行

任务分为几个stage，每个stage内有各自的状态，最终任务有finished状态

- mplus  提交M+任务、检查任务状态、下载文件
	- todo
	- running
	- success
	- failed
- mixreach   提交mixreach任务、检查状态、下载文件（补充tv数据要在这个阶段）
- merge
### 4.3 重运行

选择stage重运行，修改stage、status为todo
不选择，修改stage=mplus、status为todo

