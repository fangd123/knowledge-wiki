---
title: "《聊天框之外：AI交互设计小课》系列设定 v7b"
date: 2026-11-08
author: fangd123
license: CC BY-NC-SA 4.0
category: 产品方法论
series: AI交互设计科普视频策划
series_order: 5
tags:
  - 产品方法论
  - AI交互设计科普视频策划
ai_assisted: true
status: review
---

# 《聊天框之外：AI交互设计小课》系列设定 v7b

更新：2026-11-08。最高依据为《总体规划.md》v2、《系列大纲v2.md》、《风格指导.md》v1.2；本文件是执行规格，冲突时服从以上三份文件。观众是对AI感兴趣、未必熟悉设计术语的普通人。目的：从生活任务中看清交互方式的选择，能够检查常用AI功能。

三幕共八集：EP01–03「卡在哪」；EP04–05「谁走过这条路」；EP06–08「怎么做」。一篇文章不再对应一集；原文A01–A05的检索映射保留在每集稿末。机票案例从提出、表达负担、并排比较，走到可控调整和提交前确认。每集聊天适用场景集中在一处讲2句，位置与对白检查沿用自检_v6，本轮画面引导语最小修订见自检_v7；历史和作者框架归属留在来源区。

|集|观众问题|页数|汉字数|时长预算|
|---:|---|---:|---:|---|
|1|AI为什么长成了聊天框？|14|1216|5:30|
|2|跟AI说话，为什么这么累？|18|1654|7:00|
|3|聊天框里，为什么摆不开一张工作台？|17|1698|7:00|
|4|最好的技术，为什么你注意不到？|21|1619|7:30|
|5|为什么一根手指胜过一句命令？|19|1661|7:00|
|6|AI能不能不等我开口？|19|1626|7:00|
|7|能不能直接“捏”AI的输出？|16|1684|7:00|
|8|AI越主动，越要握住什么？|24|1767|8:33|

时长为分镜预算，包含阅读、点击及停顿，尚无本人录音实测。前15秒直接给生活钩子；标识叠在最初1–2秒，不单独做长片头。地图约3秒；通常收尾7秒，EP08最终驾驶画面10秒；历史解释、问题衔接放在收尾前一页。每次点击揭示一个意思，平均约2次/页；复杂页最多4次。

## 画布、文字与配色

画布13.333×7.5英寸，16:9，正式PNG/视频1920×1080，封面缩略图1280×720。左右边距0.55英寸；顶栏y=0–0.85，标题y=0.85–1.75，内容y=1.85–6.35，y≥6.525为完全空白字幕安全区。底部没有面包屑或进度导航。

|用途|字体|字号|
|---|---|---|
|封面问题|得意黑|66–80pt；样片66pt|
|页标题|得意黑|40pt；≤16字|
|金句|得意黑|54–64pt|
|卡片标题/短标签|思源黑体 CN Medium|26–30pt|
|短句|思源黑体 CN|22–24pt|
|气泡、旁批|霞鹜文楷|22–26pt；气泡≤12字|
|系列名|思源黑体 CN|14pt，风格指导明确指定的顶栏例外|
|来源、照片年份和署名|思源黑体 CN|10–12pt；按照片规范|

其余文字最小18pt，常驻文字不含标题和来源≤40汉字。字号不足时重排或改短。字体显式写入latin/ea/cs三组，不依赖主题回退；字体不嵌入PPTX。OFL字体和许可留在`_assets/fonts/`。

|语义|色值|
|---|---|
|纸面/白卡/墨线|#FFF8E7 / #FFFFFF / #25324B|
|人、痛点|#FF9B54，浅底#FFE9D6|
|AI|#4A7CFF，浅底#E3EBFF|
|更好、可控|#49A88F，浅底#DFF3EC|
|荧光笔/腮红/灰|#FFE27A / #FFB3A7 / #8A93A6|
|大卡片硬影|#EADFC6|

每页最多两种强调色，对照页左橙右绿。顶栏定位点用墨色，避免为对照页另引入蓝色；双角色页用橙蓝；按第4阶段要求，封面固定黄色荧光笔为明确例外，左下第N集贴纸得意黑44pt。EP01 P05另允许#B9825A皮革色配白色虚线缝线。收尾便签用浅蓝。荧光笔组件仍保留标准黄色，组件样张P13展示。照片本身的色彩、肤色与固定腮红不作为额外的语义强调。

## 图形与角色

角色、卡片、气泡、贴纸2.25pt墨线；界面内部控件1.5pt；小图标1.75pt。硬影复制同一preset向右下偏移0.07英寸，无模糊；大卡片米色、小元素墨色。所有图形、线、组、文本清空主题effectRef和effectLst，避免Office主题软阴影。便签顶部加胶带，贴纸有白边，旋转±3°；手机、聊天窗、面板不旋转。每页贴纸/便签≤3、涂鸦≤2。

小问头占总高约40%，橙T恤、粗圆角条四肢、墨裤圆头鞋、三笔呆毛、腮红。内容页2.2–3英寸，封面3.5–4.5英寸。点点为白色屏幕脸，蓝色角标固定右上、天线和两只蓝色小手，1.6–2.2英寸。六种表情：平静、疑惑、惊讶、开心、苦恼、灵光一闪；小问六姿态stand/point/think/shrug/cheer/hold，表情与姿态可交叉，face支持朝向。

角色在提问、转折、结论页出现，约一半页面，不再要求每页出场。EP01出场7/14页，EP02出场12/18页；同位置表情两个版本计一处出场。角色不挡内容，面向所反应的内容。前3集小问在边上、点点更居中；后续分镜让点点退后、小问动手，结尾小问驾驶、点点指路。

## kit公共入口

代码以`_lib/kit.py`为公共入口，前两集场景在`_lib/scenes.py`，EP03–05在`_lib/scenes_v5.py`，EP06–08在`_lib/scenes_v6.py`；新组件分别在components_v5.py/components_v6.py，生成内容在`_lib/course.json`；口播主稿在`_lib/narration_v4.txt`，结构/视觉主数据在`_tools/course_v4_setup.py`，第5阶段合并在`_tools/course_v5_setup.py`，第6阶段修订与三组合并在`_tools/course_v6_setup.py`；第7阶段点名页与4页口播引导修订在`_tools/course_v7_setup.py`，实际点击/角色元数据在`_lib/visual_v7.json`，由`write_course.py`统一导出。EP01–08的build.py只导入kit作为业务模块，sys/pathlib用于定位项目。

|用途|现行API|
|---|---|
|页型|cover / hook / map / compare / statement / diagram / steps / outro，签名(prs,title,ep=1)|
|角色|draw_xiaowen(slide,x,y,h,mood,pose,face)；draw_diandian(slide,x,y,h,mood,face)|
|气泡/云|bubble(slide,x,y,w,h,value,side='user',size=24,target=None)；thought_cloud(slide,x,y,w,h,value='',target=None)|
|地图|map_component(slide,x,y,w=11.6,current=3,final=False)，current按0–4索引|
|照片|photo_card(slide,img_path,name,year,credit,x,y,w,rotation=-3)|
|面板/贴纸/便签|panel / sticker / sticky；英寸定位；一组图文用bundle组合|
|示意控件|phone / chat_window / slider / button / check_options；均为编辑图形，并非运行中的产品|
|强调|highlight标准黄；doodle支持star/rays/bang/question/underline/spiral/sweat/heart/check/cross|
|动画/保存|click_sequence / switch / notes / save_deck / build_episode|

气泡是`wedgeRoundRectCallout`、云是`cloudCallout`，target按靠近角色的一侧约束尾尖与尾根后映射到adjustments，不另拼三角形；尾尖延伸0.34in，尾根≥0.25in。`photo_card`等比完整容纳照片，白宽边、底边留名和年份、上贴胶带；署名在卡旁保留。真实照片仅按台账使用，封面无真人，演示操作仍自绘并标“概念示意”。

```python
from _lib import kit
p=kit.new_deck()
s=kit.hook(p,'怎样更轻松？',1)
old=kit.draw_xiaowen(s,1,3.5,2.5,'开心')
new=kit.draw_xiaowen(s,1,3.5,2.5,'苦恼')
kit.click_sequence(s,[kit.switch(old,new)])
kit.notes(s,'我期待它记住条件。[点击]忘记了，我就失望。')
kit.save_deck(p,kit.ROOT/'_review/示例.pptx')
```

淡入/淡出0.3秒，出现1毫秒。click_sequence外层每项是一次单击，同组后续效果用withEffect。`switch(old,new)`把旧角色淡出和新角色淡入合成一次点击。**效果数不等于点击数**：EP01共26次点击/26个效果，EP02共34次点击/36个效果；EP02 P14、P15各2次点击/3个效果。原始交付PPT保留两层，不能为静态截图把旧角色删除。

## 来源与验证边界

史实备注有[Sources]块，照片有《素材授权清单.md》及registry.json。Gibson提出affordance，Norman引入设计；意符与示能的术语区别留来源区，口播只讲门把手的推拉线索。Shneiderman于1983年讨论直接操控；2007 iPhone是历史产品案例。1991普适计算与后来的平静技术分开；五原则与实体虚拟化的当代延伸均为作者综合。

复建顺序见《制作手册.md》。八集均有原生PPT。EP01/02批准版哈希未变；EP03–05 v5由统筹审图整体通过，v6仅改指定五页。EP06/07/08 v6为19/16/24页，口播1629/1683/1766汉字。组件样张53页，新增18组件各一页。全部9份PPT、201页结构检查通过；空childTnLst/bldLst均为0。EP03–08与样张169页已辅助渲染，真实Office v6打开/导出和F5仍由统筹执行。历史实机证据完整保留。

本轮口播规则以《风格指导》第12节为准：短句、对白、看图提示、不念补丁。托腮拳头位于下巴下方，前臂弯折且最后绘制；手持为身体侧边小手机，高度为头径0.8倍。各集封面有独立道具和情绪，EP03–08详见分镜。

第5阶段新增API：`signpost` / `mural` / `straw_view` / `flight_card` / `workbench` / `factory` / `car_cutaway` / `bicycle` / `room` / `scale_devices` / `touch_hand` / `photo_zoom` / `screen_mode` / `toggle` / `rebound` / `desk_lamp`。样张P20–35顺序一致。EP03=33点击/39效果，EP04=41/44，EP05=37/43；每页1–4点击。

第6阶段：click_sequence无效果时完全不写timing/bldLst；validate.py默认检查八集与样张，任何空childTnLst均报错。常驻汉字按各单击可见状态峰值≤40。EP06=37点击/39效果，EP07=33/41，EP08=45/48。新增18组件的API与样张位置见自检_v6。EP08 P16保留唯一一句五条清单归属作者归纳。

## 第7阶段执行状态

EP01–05交付文件按禁改要求字节一致；EP06–08仅34点名页改变，25页XML保持；黏土几何顺序、吊挂刹车、相连L椅与7样张已修。全部逐页点击/效果/预算不变。现行112页检查通过；全量201页新增几何扫描仅报受保护EP03 P01/P02两处历史问题，源码及样张已修但未重建EP03。v7真实Office导出待统筹。

## 第7b阶段状态（覆盖前述EP03禁改约束）

统筹明确授权EP03 P01/P02重建，历史两处纸罩顺序问题已消除；EP07仅封面调整道具与人物位置。全量201页0错误，口播、逐页点击/效果不变，EP03点击清单字节不变。EP01/02/04/05/06/08及kit不重建。辅助逐像素仅3指定页变化；真实Office启动报0x80070520，真实像素比对待桌面运行export_png_v7b.ps1。详情自检_v7b。
