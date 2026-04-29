# 原文
> https://addyosmani.com/blog/21-lessons/

21 Lessons From 14 Years at Google
January 3, 2026
When I joined Google ~14 years ago, I thought the job was about writing great code. I was partly right. But the longer I’ve stayed, the more I’ve realized that the engineers who thrive aren’t necessarily the best programmers - they’re the ones who’ve figured out how to navigate everything around the code: the people, the politics, the alignment, the ambiguity.

These lessons are what I wish I’d known earlier. Some would have saved me months of frustration. Others took years to fully understand. None of them are about specific technologies - those change too fast to matter. They’re about the patterns that keep showing up, project after project, team after team.

I’m sharing them because I’ve benefited enormously from engineers who did the same for me. Consider this my attempt to pay it forward.

1. The best engineers are obsessed with solving user problems.
It’s seductive to fall in love with a technology and go looking for places to apply it. I’ve done it. Everyone has. But the engineers who create the most value work backwards: they become obsessed with understanding user problems deeply, and let solutions emerge from that understanding.

User obsession means spending time in support tickets, talking to users, watching users struggle, asking “why” until you hit bedrock. The engineer who truly understands the problem often finds that the elegant solution is simpler than anyone expected.

The engineer who starts with a solution tends to build complexity in search of a justification.

2. Being right is cheap. Getting to right together is the real work.
You can win every technical argument and lose the project. I’ve watched brilliant engineers accrue silent resentment by always being the smartest person in the room. The cost shows up later as “mysterious execution issues” and “strange resistance.”

The skill isn’t being right. It’s entering discussions to align on the problem, creating space for others, and remaining skeptical of your own certainty.

Strong opinions, weakly held - not because you lack conviction, but because decisions made under uncertainty shouldn’t be welded to identity.

3. Bias towards action. Ship. You can edit a bad page, but you can’t edit a blank one.
The quest for perfection is paralyzing. I’ve watched engineers spend weeks debating the ideal architecture for something they’ve never built. The perfect solution rarely emerges from thought alone - it emerges from contact with reality. AI can in many ways help here.

First do it, then do it right, then do it better. Get the ugly prototype in front of users. Write the messy first draft of the design doc. Ship the MVP that embarrasses you slightly. You’ll learn more from one week of real feedback than a month of theoretical debate.

Momentum creates clarity. Analysis paralysis creates nothing.

4. Clarity is seniority. Cleverness is overhead.
The instinct to write clever code is almost universal among engineers. It feels like proof of competence.

But software engineering is what happens when you add time and other programmers. In that environment, clarity isn’t a style preference - it’s operational risk reduction.

Your code is a strategy memo to strangers who will maintain it at 2am during an outage. Optimize for their comprehension, not your elegance. The senior engineers I respect most have learned to trade cleverness for clarity, every time.

5. Novelty is a loan you repay in outages, hiring, and cognitive overhead.
Treat your technology choices like an organization with a small “innovation token” budget. Spend one each time you adopt something materially non-standard. You can’t afford many.

The punchline isn’t “never innovate.” It’s “innovate only where you’re uniquely paid to innovate.” Everything else should default to boring, because boring has known failure modes.

The “best tool for the job” is often the “least-worst tool across many jobs”-because operating a zoo becomes the real tax.

6. Your code doesn’t advocate for you. People do.
Early in my career, I believed great work would speak for itself. I was wrong. Code sits silently in a repository. Your manager mentions you in a meeting, or they don’t. A peer recommends you for a project, or someone else.

In large organizations, decisions get made in meetings you’re not invited to, using summaries you didn’t write, by people who have five minutes and twelve priorities. If no one can articulate your impact when you’re not in the room, your impact is effectively optional.

This isn’t strictly about self-promotion. It’s about making the value chain legible to everyone- including yourself.

7. The best code is the code you never had to write.
We celebrate creation in engineering culture. Nobody gets promoted for deleting code, even though deletion often improves a system more than addition. Every line of code you don’t write is a line you never have to debug, maintain, or explain.

Before you build, exhaust the question: “What would happen if we just… didn’t?” Sometimes the answer is “nothing bad,” and that’s your solution.

The problem isn’t that engineers can’t write code or use AI to do so. It’s that we’re so good at writing it that we forget to ask whether we should.

8. At scale, even your bugs have users.
With enough users, every observable behavior becomes a dependency - regardless of what you promised. Someone is scraping your API, automating your quirks, caching your bugs.

This creates a career-level insight: you can’t treat compatibility work as “maintenance” and new features as “real work.” Compatibility is product.

Design your deprecations as migrations with time, tooling, and empathy. Most “API design” is actually “API retirement.”

9. Most “slow” teams are actually misaligned teams.
When a project drags, the instinct is to blame execution: people aren’t working hard enough, the technology is wrong, there aren’t enough engineers. Usually none of that is the real problem.

In large companies, teams are your unit of concurrency, but coordination costs grow geometrically as teams multiply. Most slowness is actually alignment failure - people building the wrong things, or the right things in incompatible ways.

Senior engineers spend more time clarifying direction, interfaces, and priorities than “writing code faster” because that’s where the actual bottleneck lives.

10. Focus on what you can control. Ignore what you can’t.
In a large company, countless variables are outside your control - organizational changes, management decisions, market shifts, product pivots. Dwelling on these creates anxiety without agency.

The engineers who stay sane and effective zero in on their sphere of influence. You can’t control whether a reorg happens. You can control the quality of your work, how you respond, and what you learn. When faced with uncertainty, break problems into pieces and identify the specific actions available to you.

This isn’t passive acceptance but it is strategic focus. Energy spent on what you can’t change is energy stolen from what you can.

11. Abstractions don’t remove complexity. They move it to the day you’re on call.
Every abstraction is a bet that you won’t need to understand what’s underneath. Sometimes you win that bet. But something always leaks, and when it does, you need to know what you’re standing on.

Senior engineers keep learning “lower level” things even as stacks get higher. Not out of nostalgia, but out of respect for the moment when the abstraction fails and you’re alone with the system at 3am. Use your stack.

But keep a working model of its underlying failure modes.

12. Writing forces clarity. The fastest way to learn something better is to try teaching it.
Writing forces clarity. When I explain a concept to others - in a doc, a talk, a code review comment, even just chatting with AI - I discover the gaps in my own understanding. The act of making something legible to someone else makes it more legible to me.

This doesn’t mean that you’re going to learn how to be a surgeon by teaching it, but the premise still holds largely true in the software engineering domain.

This isn’t just about being generous with knowledge. It’s a selfish learning hack. If you think you understand something, try to explain it simply. The places where you stumble are the places where your understanding is shallow.

Teaching is debugging your own mental models.

13. The work that makes other work possible is priceless - and invisible.
Glue work - documentation, onboarding, cross-team coordination, process improvement - is vital. But if you do it unconsciously, it can stall your technical trajectory and burn you out. The trap is doing it as “helpfulness” rather than treating it as deliberate, bounded, visible impact.

Timebox it. Rotate it. Turn it into artifacts: docs, templates, automation. And make it legible as impact, not as personality trait.

Priceless and invisible is a dangerous combination for your career.

14. If you win every debate, you’re probably accumulating silent resistance.
I’ve learned to be suspicious of my own certainty. When I “win” too easily, something is usually wrong. People stop fighting you not because you’ve convinced them, but because they’ve given up trying - and they’ll express that disagreement in execution, not meetings.

Real alignment takes longer. You have to actually understand other perspectives, incorporate feedback, and sometimes change your mind publicly.

The short-term feeling of being right is worth much less than the long-term reality of building things with willing collaborators.

15. When a measure becomes a target, it stops measuring.
Every metric you expose to management will eventually be gamed. Not through malice, but because humans optimize for what’s measured.

If you track lines of code, you’ll get more lines. If you track velocity, you’ll get inflated estimates.

The senior move: respond to every metric request with a pair. One for speed. One for quality or risk. Then insist on interpreting trends, not worshiping thresholds. The goal is insight, not surveillance.

16. Admitting what you don’t know creates more safety than pretending you do.
Senior engineers who say “I don’t know” aren’t showing weakness - they’re creating permission. When a leader admits uncertainty, it signals that the room is safe for others to do the same. The alternative is a culture where everyone pretends to understand and problems stay hidden until they explode.

I’ve seen teams where the most senior person never admitted confusion, and I’ve seen the damage. Questions don’t get asked. Assumptions don’t get challenged. Junior engineers stay silent because they assume everyone else gets it.

Model curiosity, and you get a team that actually learns.

17. Your network outlasts every job you’ll ever have.
Early in my career, I focused on the work and neglected networking. In hindsight, this was a mistake. Colleagues who invested in relationships - inside and outside the company - reaped benefits for decades.

They heard about opportunities first, could build bridges faster, got recommended for roles, and co-founded ventures with people they’d built trust with over years.

Your job isn’t forever, but your network is. Approach it with curiosity and generosity, not transactional hustle.

When the time comes to move on, it’s often relationships that open the door.

18. Most performance wins come from removing work, not adding cleverness.
When systems get slow, the instinct is to add: caching layers, parallel processing, smarter algorithms. Sometimes that’s right. But I’ve seen more performance wins from asking “what are we computing that we don’t need?”

Deleting unnecessary work is almost always more impactful than doing necessary work faster. The fastest code is code that never runs.

Before you optimize, question whether the work should exist at all.

19. Process exists to reduce uncertainty, not to create paper trails.
The best process makes coordination easier and failures cheaper. The worst process is bureaucratic theater - it exists not to help but to assign blame when things go wrong.

If you can’t explain how a process reduces risk or increases clarity, it’s probably just overhead.

And if people are spending more time documenting their work than doing it, something has gone deeply wrong.

20. Eventually, time becomes worth more than money. Act accordingly.
Early in your career, you trade time for money - and that’s fine. But at some point, the calculus inverts. You start to realize that time is the non-renewable resource.

I’ve watched senior engineers burn out chasing the next promo level, optimizing for a few more percentage points of compensation. Some of them got it. Most of them wondered, afterward, if it was worth what they gave up.

The answer isn’t “don’t work hard.” It’s “know what you’re trading, and make the trade deliberately.”

21. There are no shortcuts, but there is compounding.
Expertise comes from deliberate practice - pushing slightly beyond your current skill, reflecting, repeating. For years. There’s no condensed version.

But here’s the hopeful part: learning compounds when it creates new options, not just new trivia. Write - not for engagement, but for clarity. Build reusable primitives. Collect scar tissue into playbooks.

The engineer who treats their career as compound interest, not lottery tickets, tends to end up much further ahead.

A final thought
Twenty-one lessons sounds like a lot, but they really come down to a few core ideas: stay curious, stay humble, and remember that the work is always about people - the users you’re building for and the teammates you’re building with.

A career in engineering is long enough to make plenty of mistakes and still come out ahead. The engineers I admire most aren’t the ones who got everything right - they’re the ones who learned from what went wrong, shared what they discovered, and kept showing up.

If you’re early in your journey, know that it gets richer with time. If you’re deep into it, I hope some of these resonate.

# 译文（机翻）

只要用户数量足够多，每一种可观察到的行为都会成为一种依赖——无论你之前承诺过什么。总有人在抓取你的 API，自动化处理你的特性，缓存你的漏洞。
这就带来了一个职业层面的洞见：你不能把兼容视为“维护”，而把新特性视为“真正的工作”。兼容性就是产品。
将你的弃用设计为包含时间、工具和同理心的迁移。大多数“API设计”实际上是“API退役”。
9. 大多数“慢”团队实际上是目标不一致的团队。
当一个项目进展缓慢时，人们的本能反应是指责执行不力：员工不够努力、技术选择错误、工程师数量不足。通常，这些都不是真正的问题。
在大公司里，团队是并发工作的单元，但随着团队数量的增加，协调成本呈几何级数增长。大多数的低效实际上是协作失败——人们在做错误的事情，或者以不兼容的方式做正确的事情。
高级工程师花在明确方向、接口和优先级上的时间比“更快地编写代码”更多，因为实际的瓶颈就在于此。
10. 专注于你能控制的事情，忽略你无法控制的事情。
在大公司里，无数的变量是你无法控制的——组织变革、管理决策、市场变化、产品转向。纠结于这些只会徒增焦虑，却无法改变现状。
那些保持理智且高效的工程师会专注于自己的影响力范围。你无法控制是否会发生重组。你可以控制自己工作的质量、你的应对方式以及你学到的东西。面对不确定性时，将问题拆解成小部分，并确定你可以采取的具体行动。
这不是被动接受，而是战略聚焦。把精力花在你无法改变的事情上，就是从你能改变的事情上窃力。
11. 抽象并不会消除复杂性。它们只是把复杂性转移到你值班的那天。
每一次抽象都是一场赌注，赌你不需要了解底层的东西。有时你能赢下这场赌注。但总会有东西泄露，而当这种情况发生时，你需要知道自己站在什么之上。
资深工程师即便技术栈不断攀升，也会持续学习“底层”知识。这并非出于怀旧，而是出于对抽象概念失效、凌晨三点独自面对系统这一时刻的尊重。运用你的技术栈。
但保留其潜在故障模式的工作模型。
12. 写作促使思路清晰。更好地学习某样东西的最快方法就是尝试去教授它。
写作促使思路清晰。当我向他人解释一个概念时——无论是在文档、演讲、代码审查评论中，甚至只是与AI聊天——我都会发现自己理解上的漏洞。让别人能够理解某件事的过程，也让我自己对它有了更清晰的认识。
这并不意味着你能通过教授外科手术来学会如何成为一名外科医生，但这一前提在软件工程领域大体上仍然成立。
这不仅仅是关于慷慨地分享知识，这是一种利己的学习技巧。如果你认为自己理解了某件事，那就试着简单地解释一下。你磕绊的地方，就是你理解浅薄的地方。
教学就是调试自己的心智模型。
13. 使其他工作成为可能的工作是无价的——而且是无形的。
衔接——文档编写、入职引导、跨团队协调、流程改进——至关重要。但如果无意识地去做，它可能会阻碍你的技术发展轨迹，让你精疲力竭。陷阱在于将其作为“乐于助人”的行为来做，而不是将其视为有计划、有边界、有可见影响力的工作。
设定时限。循环利用。将其转化为成果：文档、模板、自动化。并使其以影响力而非个人特质的形式清晰呈现。
无价且无形，这对你的职业生涯来说是一种危险的组合。
14. 如果你在每场辩论中都获胜，你可能正在积累无声的抵抗。
我已经学会对自己的笃定保持怀疑。当我“太轻易”获胜时，通常就有问题了。人们不再与你争论，不是因为你说服了他们，而是因为他们已经放弃尝试——他们会在执行中而非会议上表达这种分歧。
真正的协调需要更长的时间。你必须真正理解其他观点，吸收反馈，有时还要公开改变自己的想法。
短期的正确感远不如与积极的合作者共同建设的长期现实有价值。
15. 当一项指标成为目标时，它就不再是有效的衡量标准了。
你向管理层展示的每一项指标最终都会被钻空子。这并非出于恶意，而是因为人们会针对被衡量的内容进行优化。
如果你追踪代码行数，你会得到更多的代码行。如果你追踪速度，你会得到夸大的估算。
资深人士的做法：对每一项指标要求都用一对指标来回应。一个关乎速度，一个关乎质量或风险。然后坚持解读趋势，而非盲目崇拜阈值。目标是洞察，而非监控。
16. 承认自己不知道比假装知道更能带来安全感。
那些敢于说“我不知道”的资深工程师并非在示弱——他们是在创造一种许可。当领导者承认自己的不确定性时，这表明在这个环境中，其他人也可以这样做。反之，就会形成一种人人都假装理解，问题被隐藏起来，直到爆发的文化。
我见过这样的团队，其中最资深的人从不承认自己有困惑，我也见过由此造成的危害。问题无人提出，假设无人质疑。初级工程师保持沉默，因为他们以为其他人都懂。
树立好奇心的榜样，你就会拥有一个真正学习的团队。
17. 你的人脉比你从事过的任何工作都要长久。
在我职业生涯早期，我专注于工作，却忽视了人脉拓展。事后看来，这是个错误。那些在公司内外都注重人际关系投资的同事，几十年来都从中受益。
他们首先听说了机会，能够更快地搭建人脉桥梁，获得角色推荐，并与多年来建立信任的人共同创立企业。
工作并非永恒，但人脉却是。以好奇和慷慨之心对待人脉，而非功利性的钻营。
当到了继续前行的时候，往往是人际关系打开了那扇门。
18. 大多数性能提升来自于减少工作量，而非增加巧妙性。
当系统变慢时，本能反应是增加：缓存层、并行处理、更智能的算法。有时这样做是对的。但我发现，通过问“我们在计算哪些不需要的东西？”能获得更多性能提升。
删除不必要的工作几乎总是比更快地完成必要的工作更有成效。最快的代码是永远不会运行的代码。
在优化之前，先问问这项工作是否真的有必要存在。
19. 流程的存在是为了减少不确定性，而不是为了制造书面记录。
最佳流程能让协作更轻松，让失败的代价更低。最糟糕的流程则是官僚主义的闹剧——它的存在不是为了提供帮助，而是在事情出错时推卸责任。
如果你无法解释一个流程如何降低风险或提高清晰度，那它可能只是额外负担。
如果人们花在记录工作上的时间比实际工作的时间还多，那就说明出了大问题。
20. 最终，时间比金钱更有价值。请据此行事。
在职业生涯早期，你用时间换取金钱——这没问题。但到了某个阶段，情况就反过来了。你开始意识到，时间才是不可再生的资源。
我见过资深工程师为了追求下一个晋升级别、为了多争取几个百分点的薪酬而心力交瘁。其中一些人如愿以偿。但大多数人后来都在想，这是否值得他们所放弃的一切。
答案不是“不要努力工作”，而是“清楚自己在做什么交易，并有意为之”。
21. 没有捷径，但有复利。
专业能力源自刻意练习——稍稍超越你当前的技能水平，进行反思，然后不断重复。这样坚持数年。没有捷径可走。
但这里有充满希望的一面：当学习创造新的选择，而不只是新的琐事时，它就会产生复利效应。写作——不是为了吸引眼球，而是为了清晰明了。构建可复用的基本要素。将经验教训整理成行动手册。
那些将自己的职业生涯视为复利而非彩票的工程师，往往最终会取得更大的进步。
最后的思考
二十一个课程听起来很多，但实际上归结为几个核心思想：保持好奇心，保持谦逊，并记住工作始终围绕着人——你为之构建产品的用户以及与你一同构建的队友。
工程领域的职业生涯足够漫长，即便犯了很多错误，仍能取得成功。我最钦佩的工程师并非那些事事都做对的人，而是那些从错误中学习、分享自己的发现，并始终坚持前行的人。
如果你才刚刚踏上征程，要知道它会随着时间的推移而更加丰富。如果你已经深入其中，我希望其中一些能引起你的共鸣。