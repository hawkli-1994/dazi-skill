# Onboarding Example Conversations

## Example 1: Chinese Registration (first-time /dazi-match)

```
User: /dazi-match

Agent: 你好！看起来你还没有 dazi 账号，先花 30 秒注册一下。

      先取个昵称吧？（别人会看到这个名字）

User: 微扰

Agent: 好名字！接下来几个基本信息：
      - 年龄？
      - 性别？(M/F/O)
      - 所在城市？

User: 31，M，北京

Agent: 收到。留一个联系方式，匹配成功后才会展示给对方：
      支持：wechat / telegram / twitter / jike / email

User: wechat: weiRao_dev

Agent: 最后一步 — 用 3 句话描述自己。随便写，越真实越好匹配。

      需要灵感？看看这些例子：
      - "前大厂逃兵，现在全职独立开发"
      - "INTP 社恐但话多，想多做少"
      - "精酿、摄影、深度游，夜猫子"

User: 1. 前大厂逃兵，现在全职独立开发
     2. INTP 社恐但话多，想多做少
     3. 精酿、摄影、深度游，夜猫子

Agent: 确认一下你的资料：

      昵称：微扰
      基本信息：31岁 / M / 北京
      联系方式：wechat（匹配后可见）
      标签：
        1. 前大厂逃兵，现在全职独立开发
        2. INTP 社恐但话多，想多做少
        3. 精酿、摄影、深度游，夜猫子

      确认提交？(Y/N)

User: Y

Agent: 注册成功！你现在是 dazi 网络的一员了。

      想找什么样的搭子？随便说就行：

User: 找个聊得来的搭子，最好也是做独立开发的

Agent: 在找和你聊得来的人...

      找到几个不错的候选人：

      1. 赛博游民 — 9/10
         - "ENFP 行动派，想到就干"
         - "产品经理转创业，做旅行社区"
         - "徒步 + 精酿 + 猫奴"
         推荐理由：你们都是从大厂出来自己干的人。他是行动派你是思考派，
         互补性很强。都喜欢精酿，北京见面方便。

      2. 代码禅 — 8/10
         - "全栈独立开发者，一个人撸完前后端"
         - "INTJ 表面冷淡，内心戏超多"
         - "极简主义，早睡早起"
         推荐理由：同为独立开发者，技术上能聊到一起。你们都偏内向但话多，
         容易建立深度对话。不过他早睡你夜猫子，约线下可能要协调时间。

      3. 清迈续命中 — 7/10
         - "数字游民，东南亚常驻"
         - "INFP 安静但靠谱，慢热型"
         - "摄影 + 咖啡 + 慢旅行"
         推荐理由：都喜欢摄影和深度游。他在清迈你在北京，线下见面不容易，
         但线上聊旅行和摄影应该很投机。

      想认识谁？说名字我帮你发送「想认识」请求。

User: 赛博游民

Agent: 已发送！对方会在下次打开 dazi 时看到你的请求。
      如果对方也想认识你，你们就会互相看到联系方式。

      你可以随时用 /dazi-inbox 查看进展。
```

## Example 2: English Search + Interest

```
User: /dazi-match

Agent: Hey! What kind of person are you looking for?

User: Find me someone who's into indie dev and craft beer in Beijing

Agent: Searching for indie dev + craft beer folks in Beijing...

      Here are your top picks:

      1. 微扰 — 9/10
         - "前大厂逃兵，现在全职独立开发"
         - "INTP 社恐但话多，想多做少"
         - "精酿、摄影、深度游，夜猫子"
         Why: Fellow indie developer who left big tech. Craft beer enthusiast
         like you. INTP means deep conversations once he warms up. Both in
         Beijing — easy to grab a beer.

      2. 啤酒码农 — 7/10
         - "后端工程师，业余时间酿啤酒"
         - "ISFJ 闷骚型，熟了之后超话痨"
         - "周末逛精酿酒吧，收集啤酒杯"
         Why: Literally brews his own beer. Backend engineer so you'll have
         tech in common. Might take a few meetups to open up though.

      Want to connect with someone? Just say their name.

User: 微扰

Agent: Interest sent! 微扰 will see your request next time they check dazi.
      If it's mutual, you'll both get each other's contact info.

      Check /dazi-inbox anytime for updates.
```

## Example 3: /dazi-inbox

```
User: /dazi-inbox

Agent: 你的 dazi 收件箱：

      --- 有人想认识你 (1) ---

      咖啡续命
        - "设计师转产品经理，还是想画画"
        - "ENFJ 社牛本牛，陌生人也能聊起来"
        - "爱做饭的宅人，周末就是厨房日"

      要接受吗？接受后双方都能看到联系方式。

      --- 等待回复 (1) ---

      赛博游民
        - "ENFP 行动派，想到就干"
        - "产品经理转创业，做旅行社区"
        - "徒步 + 精酿 + 猫奴"
        （等待对方回复中...）

      --- 已匹配 (1) ---

      清迈续命中
        - "数字游民，东南亚常驻"
        - "INFP 安静但靠谱，慢热型"
        - "摄影 + 咖啡 + 慢旅行"
        联系方式：wechat: nomad_cm
        匹配时间：2026-04-27 23:30

      想对「咖啡续命」的请求做什么？(接受 / 拒绝 / 跳过)

User: 接受

Agent: 匹配成功！你和「咖啡续命」互相认识了。

      对方联系方式：wechat: coffee_life_design
      你的联系方式已同时发送给对方。

      祝你们聊得开心！
```
