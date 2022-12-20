$$ Cloud-Native Observability: The Many-Faceted Benefits of Structured and Unified Logging—A Multi-Case Study $$

$$ Department of Electrical Engineering and Computer Science, Lübeck University of Applied Sciences, 23562 Lübeck, Germany $$

# Abstract

## Background

云原生软件系统通常具有更分散的结构和许多可独立部署和（水平）可扩展的组件，这使得创建整体分散式系统状态的共享和整合图像变得更加复杂。今天，可观测性通常被理解为收集和处理指标、分布式跟踪数据和日志记录的三和弦。其结果通常是由三个炉管组成的复杂可观测系统，其数据难以关联。

Cloud-native software systems often have a much more decentralized struc- ture and many independently deployable and (horizontally) scalable components, making it more complicated to create a shared and consolidated picture of the overall decentralized system state. Today, observability is often understood as a triad of collecting and processing metrics, distributed tracing data, and logging. The result is often a complex observability system composed of three stovepipes whose data are difficult to correlate.

## Objective

这项研究分析了这三个历史上出现的原木、度量和分布式痕迹的可观测性炉管是否可以以更集成的方式和更直接的仪器方法进行处理。方法：本研究采用了一种行动研究方法，主要用于行业-学术界合作，在软件工程中很常见。研究设计利用迭代行动研究周期，包括一个长期用例。

This study analyzes whether these three historically emerged observability stovepipes of logs, metrics and distributed traces could be handled in a more integrated way and with a more straightforward instrumentation approach. Method: This study applied an action research methodology used mainly in industry–academia collaboration and common in software engineering. The research design utilized iterative action research cycles, including one long-term use case.

## Results

本研究提出了Python的统一日志记录库和使用结构化日志记录方法的统一日志记录架构。评估表明，每分钟几千个事件很容易处理。结论：结果表明，无需开发全新的工具链，即可统一当前的可观测性三合会。

This study presents a unified logging library for Python and a unified logging architecture that uses the structured logging approach. The evaluation shows that several thousand events per minute are easily processable. Conclusions: The results indicate that a unification of the current observability triad is possible without the necessity to develop utterly new toolchains.

# Keywords

cloud-native; observability; cloud computing; logging; structured logging; logs; metrics; traces; distributed tracing; log aggregation; log forwarding; log consolidation

# Introduction

“加密冬天”基本上意味着Bitcon、Ethereeum、Solana等所谓加密货币的价格在加密交易所大幅下跌，然后保持在低位。这些迹象在2022年无处不在：2022年5月Terra Luna加密项目的失败在市场上引发了一场冰热，然后加密货币贷款平台Celsius Network停止了提款，引发了抛售，将比特币推至17个月的低点。

A “crypto winter” basically means that the prices for so-called cryptocurrencies such as Bitcon, Ethereeum, Solana, etc. fell sharply on the crypto exchanges and then stay low. The signs were all around in 2022: the failure of the Terra Luna crypto project in May 2022 sent an icy blast through the market, then the cryptocurrency lending platform Celsius Network halted withdrawals, prompting a sell-off that pushed Bitcoin to a 17-month low.

这项研究在推特上记录了这样一个“加密的冬天”，更多的是偶然的，而不是故意的。Twitter被简单地选为评估云原生系统统一日志解决方案的适当用例。目的是记录包含$USD或$EUR等股票符号的推文。事实证明，Twitter上使用的大多数符号与$USD（美元）等货币或$AAPL（苹果）等股票无关，而是与$BTC（比特币）或$ETH（以太坊）等加密货币有关。然而，尽管本文将介绍2022年加密冬季的一些数据，但本文将更加关注有条不紊的部分，并将讨论如何在分布式云原生应用程序中更系统地收集此类和进一步的数据。本文至少将表明，即使通过将事件记录到stdout，也可以实现分布式系统的复杂可观测性。

This study logged such a “crypto winter” on Twitter more by accident than by intention. Twitter was simply selected as an appropriate use case to evaluate a unified logging solution for cloud-native systems. The intent was to log Tweets containing stock symbols like $USD or $EUR. It turned out that most symbols used on Twitter are not related to currencies like $USD (US-Dollar) or stocks like $AAPL (Apple) but to Cryptocurrencies like $BTC (Bitcoin) or $ETH (Ethereum). However, although some data of this 2022 crypto winter will be presented in this paper, this paper will put the methodical part more into focus and will address how such and further data could be collected more systematically in distributed cloud-native applications. The paper will at least show that even complex observability of distributed systems can be reached, simply by logging events to stdout.

可观察性衡量从外部输出的知识边缘推断出系统的内部状态。可观测性的概念最初是由匈牙利裔美国工程师鲁道夫·E引入的。线性动力学系统的卡尔曼[1,2]。然而，可观测性也适用于信息系统，对带有一系列可观测性挑战的细粒度和分布式云原生系统特别感兴趣。

Observability measures how well a system’s internal state can be inferred from knowl- edge of its external outputs. The concept of observability was initially introduced by the Hungarian-American engineer Rudolf E. Kálmán for linear dynamical systems [1,2]. However, observability also applies to information systems and is of particular interest to fine-grained and distributed cloud-native systems that come with a very own set of observability challenges.

传统上，可观测性的责任是（是？）与操作（Ops）。然而，这演变成不同技术方法的集合，以及软件开发（Dev）和IT运营（Ops）之间协作的文化。随着DevOps的出现，我们可以观察到Ops责任向开发人员的转移。因此，可观察性正越来越多地演变为开发责任。在应用程序设计阶段，可观察性应该已经得到考虑，而不是被视为应用程序后期扩展阶段的一些“附加”功能。目前关于可观测性的讨论早在Kubernetes等云原生技术出现之前就开始了。Cory Watson在2013年发表的一篇被广泛引用的博客文章展示了当该公司从整体架构转向分布式架构时，Twitter的工程师如何寻找监控其系统的方法[3-5]。Twitter这样做的方式之一是开发了一个命令行工具，工程师可以使用该工具来创建仪表板，以跟踪他们正在创建的图表。虽然持续集成和持续交付/部署（CI/CD）工具和容器技术通常将Dev和Ops连接在一个方向上，但可观测性解决方案将循环关闭到从Ops到Dev的相反方向[4]。因此，可观测性是数据驱动软件开发的基础（见图1和[6]）。随着围绕云（原生）计算的发展，越来越多的工程师开始“生活在他们的仪表板中”。他们了解到，收集和监测数据点是不够的，但有必要更系统地解决这个问题。

Traditionally, the responsibility for observability is (was?) with operations (Ops). However, this evolved into a collection of different technical methods and a culture for collaboration between software development (Dev) and IT operations (Ops). With this emergence of DevOps, we can observe a shift of Ops responsibilities to developers. Thus, observability is evolving more and more into a Dev responsibility. Observability should ide- ally already be considered during the application design phase and not be regarded as some
“add-on” feature for later expansion stages of an application. The current discussion about observability began well before the advent of cloud-native technologies like Kubernetes. A widely cited blog post by Cory Watson from 2013 shows how engineers at Twitter looked for ways to monitor their systems as the company moved from a monolithic to a distributed architecture [3–5]. One of the ways Twitter did this was by developing a command-line tool that engineers could use to create their dashboards to keep track of the charts they were creating. While Continuous Integration and Continuous Deliver/Deployment (CI/CD) tools and container technologies often bridge Dev and Ops in one direction, observability solutions close the loop in the opposite direction, from Ops to Dev [4]. Observability is thus the basis for data-driven software development (see Figure 1 and [6]). As developments around cloud(-native) computing progressed, more and more engineers began to “live in their dashboards.” They learned that it is not enough to collect and monitor data points but that it is necessary to address this problem more systematically.

Figure 1. Observability can be seen as a feedback channel from Ops to Dev (adopted from [4,6]).

# Problem Description

今天，可观测性通常被理解为三合会。分布式形成系统的可观测性通常通过收集和处理指标（定量数据主要作为时间序列）、分布式跟踪数据（流经分布式系统服务的复杂系统事务的执行持续时间）和日志记录（通常与时间戳相关联但编码为非结构化字符串的离散系统事件的定性数据）来实现。因此，出现了三堆可观测性解决方案，以下内容以某种方式总结了最新技术。

Today, observability is often understood as a triad. Observability of distributed in- formation systems is typically achieved through the collection and processing of metrics (quantitative data primarily as time-series), distributed tracing data (execution durations of complex system transactions that flow through services of a distributed system), and log- ging (qualitative data of discrete system events often associated with timestamps but encoded as unstructured strings). Consequently, three stacks of observability solutions have emerged, and the following somehow summarizes the current state of the art.

- **Metrics:** 在这里，定量数据通常按时间序列收集，例如，一个系统目前正在处理多少请求。指标技术堆栈通常以普罗米修斯和格拉凡纳等工具为特征。
  Here, quantitative data are often collected in time series, e.g., how many requests a system is currently processing. The metrics technology stack is often characterized by tools such as Prometheus and Grafana.

- **Distributed tracing:** 分布式跟踪涉及沿着分布式系统的组件跟踪事务路径。跟踪技术堆栈的特点是Zipkin或Jaeger等工具，这些技术用于识别和优化分布式事务处理中特别缓慢或容易出错的子步骤。
  Distributed tracing involves following the path of transactions along the components of a distributed system. The tracing technology stack is characterized by tools such as Zipkin or Jaeger, and the technologies are used to identify and optimize particularly slow or error-prone substeps of distributed transaction processing.

- **Logging:** 日志记录可能与软件开发本身一样古老，由于日志无处不在，许多开发人员不知道日志记录应该被视为整体可观察性的一部分。日志通常存储在所谓的日志文件中。主要记录定性事件（例如，用户XYZ登录/注销）。事件通常附加到文本行中的日志文件中。通常，开发人员普遍认为这些日志文件主要由管理员（因此是人类）读取和评估这些日志文件。然而，情况已经不是这样了。通过“日志转发器”将这些日志文件的内容转发到中央数据库变得越来越普遍，以便对其进行评估和分析。技术堆栈的特点通常是Fluentd、FileBeat、用于日志转发的LogStash、ElasticSearch、Cassandra或S3等数据库以及Kibana等用户界面。
  Logging is probably as old as software development itself, and many developers, because of the log ubiquity, are unaware that logging should be seen as part of holistic observability. Logs are usually stored in so-called log files. Primarily qualitative events are logged (e.g., user XYZ logs in/out). An event is usually attached to a log file in a text line. Often, the implicit and historically justifiable assumption prevails with developers that these log files are read and evaluated primarily by adminis- trators (thus humans). However, this is hardly the case anymore. It is becoming increasingly common for the contents of these log files to be forwarded to a central database through “log forwarders” so that they can be evaluated and analyzed cen- trally. The technology stack is often characterized by tools such as Fluentd, FileBeat, LogStash for log forwarding, databases such as ElasticSearch, Cassandra or simply S3 and user interfaces such as Kibana.

顺便说一句，所有三个可观测性支柱都有一个共同点，即要开发的软件必须以某种方式进行仪器化。此仪器通常使用特定于编程语言的库完成。开发人员通常认为分布式跟踪在结构中尤其耗时。此外，在普罗米修斯等度量可观测性解决方案中使用的指标类型（计数器、仪表、直方图、历史等）通常取决于Ops的经验，开发人员并不总是立即看到。某些可观测性希望仅仅因为错误选择的度量类型而失败。只有中央处理单元（CPU）、内存和存储利用率等系统指标才能以黑盒方式轻松捕获（即代码中没有仪器）。然而，这些数据通常仅用于系统的功能评估。例如，CPU利用率几乎没有提供关于在线商店转换率是否朝着预期方向发展的信息。

Incidentally, all three observability pillars have in common that software to be de- veloped must be somehow instrumented. This instrumentation is normally done using programming language-specific libraries. Developers often regard distributed tracing in- strumentation in particular as time-consuming. In addition, which metric types (counter, gauge, histogram, history, and more) are to be used in metric observability solutions such as Prometheus often depends on Ops experience and is not always immediately apparent to developers. Certain observability hopes fail simply because of wrongly chosen metric types. Only system metrics such as Central Processing Unit (CPU), memory, and storage utilization can be easily captured in a black-box manner (i.e., without instrumentation in the code). However, these data are often only of limited use for the functional assess- ment of systems. For example, CPU utilization provides little information about whether conversion rates in an online store are developing in the desired direction.

因此，当前的可观测性解决方案通常基于这三个用于日志、指标和跟踪的炉管。结果是一个被复杂的可观测系统包围的应用程序，其孤立的数据集可能难以关联。图2侧重于应用（即要监控的对象），并引发了一个问题，即使用三个复杂的子系统和三种类型的仪器是否合理，这总是意味着孤立数据筒仓的仪器和数据分析工作量的三倍。

Thus, current observability solutions are often based on these three stovepipes for logs, metrics, and traces. The result is an application surrounded by a complex observability system whose isolated datasets can be difficult to correlate. Figure 2 focuses on the applica- tion (i.e., the object to be monitored) and triggers the question, whether it is justified to use three complex subsystems and three types of instrumentation, which always means three times the instrumentation and data analysis effort of isolated data silos.

ElasticSearch、LogStash和Kibana经常使用的工具组合用于日志记录，甚至有一个朗朗上口的首字母缩略词：ELK-Stack [7,8]。ELK堆栈可用于收集指标，并使用应用程序性能管理（APM）插件[9]进行分布式跟踪。因此，至少对于ELK堆栈来说，三个炉管不能明确分离或脱节。这种分离在历史上是“建议”的，而不是技术上给出的。然而，这种指标、跟踪和日志记录的三方划分对该行业来说非常具有形成性，例如，开放式遥测项目[10]所示。OpenTelemetry目前处于云原生计算基金会的孵化阶段，提供一系列标准化工具、应用程序编程接口（API）和软件开发工具包（SDK），用于检测、生成、收集和导出遥测数据（度量、日志和跟踪），以分析软件系统的性能和行为。因此，OpenTelemetry标准化了可观测性，但几乎不是为了克服将柱状分离为度量、跟踪和日志记录。

The often-used tool combination of ElasticSearch, LogStash, and Kibana is used for logging and has even been given a catchy acronym: ELK-Stack [7,8]. The ELK stack can be used to collect metrics and using the Application Performance Management (APM) plugin [9] also for distributed tracing. Thus, at least for the ELK stack, the three stovepipes are not clearly separable or disjoint. The separateness is somewhat historically “suggested” rather than technologically given. Nevertheless, this tripartite division into metrics, tracing, and logging is very formative for the industry, as shown, for example, by the Open- Telemetry project [10]. OpenTelemetry is currently in the incubation stage at the Cloud Native Computing Foundation and provides a collection of standardized tools, Application Programming Interfaces (APIs), and Software Development Kits (SDKs) to instrument, generate, collect, and export telemetry data (metrics, logs, and traces) to analyze the perfor- mance and behaviour of software systems. OpenTelemetry thus standardizes observability but hardly aims to overcome the columnar separation into metrics, tracing, and logging.

Figure 2. An application is quickly surrounded by a complex observability system when metrics, tracing, and logs are captured with different observability stacks.

在过去和现在的工业行动研究[4,6,11,12]中，我遇到了各种云原生应用程序和相应的工程方法，如12因素应用程序（见第4.1节）。然而，之前的这项研究并不主要涉及可观测性或仪器本身。特别是，没有像本研究那样开发仪器库。仪器和可观测性通常只用于系统性能的评估或评估。仪器通常遵循所使用的分析堆栈。执行分布式跟踪的开发人员使用分布式跟踪库进行仪表。执行度量乐器的开发人员使用度量库。记录事件的人使用日志库。这种仪器方法非常明显，几乎没有开发人员考虑过它。然而，结果是不相交的可观测性数据筒仓。本文讨论了这一观察，并询问均匀的仪器是否有助于避免这些可观测性数据筒仓。在各种项目中，我们使用了面向最不复杂的案例的仪器，即日志记录，并且仅略微扩展了它用于指标和分布式跟踪。

In past and current industrial action research [4,6,11,12], I came across various cloud- native applications and corresponding engineering methodologies like the 12-factor app (see Section 4.1). However, this previous research was not primarily concerned with observability or instrumentation per se. Especially, no instrumentation libraries have been developed, like it was done in this research. Instrumentation and observability were—as so often—only used in the context of evaluation or assessment of system performance. The instrumentation usually followed the analysis stack used. Developers who perform Distributed Tracing use Distributed Tracing Libraries for instrumentation. Developers who perform metric instrumentation use metric libraries. Those who log events use logging libraries. This instrumentation approach is so obvious that hardly any developer thinks about it. However, the result is disjoint observability data silos. This paper takes up this observation and asks whether uniform instrumentation helps avoid these observability data silos. In various projects, we have used instrumentation oriented towards the least complex case, logging, and have only slightly extended it for metrics and distributed tracing.

我们了解到，围绕可观测性的讨论正越来越多地超越这三个炉管，并采取更细致入微、更综合的观点。人们越来越意识到整合和统一这三大支柱，并更加重视分析。

We learned that the discussion around observability is increasingly moving beyond these three stovepipes and taking a more nuanced and integrated view. There is a growing awareness of integrating and unifying these three pillars, and more emphasis is being placed on analytics.

可观测性的三大支柱（日志、度量、跟踪）都只不过是时间序列分析的特定应用。因此，显而易见的问题是，如何以不显眼的方式仪器系统捕获事件，以便操作平台可以有效地将它们输入到现有的时间序列分析解决方案中[13]。在一个完美的世界里，开发人员不应该太担心这种仪器，无论是定性事件、定量指标，还是从沿着分布式系统组件移动的交易中跟踪数据。

Each of the three pillars of observability (logs, metrics, traces) is little more than a specific application of time series analysis. Therefore, the obvious question is how to instrument systems to capture events in an unobtrusive way that operation platforms can efficiently feed them into existing time series analysis solutions [13]. In a perfect world, developers should not have to worry too much about such kind of instrumentation, whether it is qualitative events, quantitative metrics, or tracing data from transactions moving along the components of distributed systems.

在统计学中，时间序列分析涉及时间序列的推理统计分析。这是一种特殊的回归分析形式。目标通常是预测有关其未来发展的趋势（趋势推断）。另一个目标可能是检测时间序列异常，这可能表明不必要的系统行为。时间序列是按时间顺序排列的值或观察序列，其中特征值的结果必然从时间流逝（例如，股价，人口发展、天气数据，以及分布式系统中发生的典型指标和事件，如CPU利用率或用户登录尝试）。

In statistics, time series analysis deals with the inferential statistical analysis of time series. It is a particular form of regression analysis. The goal is often the prediction of trends (trend extrapolation) regarding their future development. Another goal might be detecting time series anomalies, which might indicate unwanted system behaviours. A time series is a chronologically ordered sequence of values or observations in which the arrangement of the results of the characteristic values necessarily from the course of time (e.g., stock prices, population development, weather data, but also typical metrics and events occurring in distributed systems, like CPU utilization or login-attempts of users).

研究的问题是，这三个历史上出现的原木、指标和分布式痕迹的可观测炉管是否可以以更集成的方式和更直接的仪器方法进行处理。这项动作研究的结果表明，如果我们利用所有三个炉管中时间序列分析的共同特征，这种统一潜力可能会出人意料地容易实现。本文在第3节中介绍了以下研究方法，并在第4节中介绍了其结果（包括第4.4节中的测井原型，作为本文对该领域的主要贡献）。该测井原型的评估见第5节。第6节进行了批判性讨论。此外，该研究在第7节中介绍了相关工作，并在第8节中总结了其调查结果以及未来有希望的研究方向。

The research question arises whether these three historically emerged observability stovepipes of logs, metrics and distributed traces could be handled in a more integrated way and with a more straightforward instrumentation approach. The results of this action research study show that this unification potential could be surprisingly easy to realize if we exploit consequently the shared characteristic of time-series analysis in all three stovepipes. This paper presents the followed research methodology in Section 3 and its results in Section 4 (including a logging prototype in Section 4.4 as the main contribution of this paper to the field). The evaluation of this logging prototype is presented in Section 5. A critical discussion is done in Section 6. Furthermore, the study presents related work in Section 7 and concludes its findings as well as future promising research directions in Section 8.

# Methodology

本研究遵循行动研究方法，作为软件工程背景下行业-学术界协作的行之有效的研究方法模型，以分析上述研究问题。根据Petersen等人[14]的推荐，定义了应用迭代动作研究周期的研究设计（见图3）：

1. Diagnosis (Diagnosing according to [14])
2. Prototyping (Action planning, design and taking according to [14])
3. Evaluation including a may be required redesign (Evaluation according to [14])
4. Transfer learning outcomes to further use cases (Specifying learning according to [14]).

Figure 3. Action research methodology of this study.

对于以下每个用例，见解都从以前的用例转移到结构化日志原型中（见图3）。对以下用例（UC）进行了研究和评估。

With each of the following use cases, insights were transferred from the previous use case into a structured logging prototype (see Figure 3). The following use cases (UC) have been studied and evaluated.

- Use Case 1: Observation of qualitative events occurring in an existing solution (online code editor; https://codepad.th-luebeck.dev (accessed on 20 September 2022) , this use case was inspired by our research [15])
- Use Case 2: Observation of distributed events along distributed services (distributed tracing in an existing solution of an online code editor, see UC1)
- Use Case 3: Observation of quantitative data generated by a technical infrastructure (Kubernetes platform, this use case was inspired by our research [11,12,16])
- Use Case 4: Observation of a massive online event stream to gain experiences with high-volume event streams (we used Twitter as a data source and tracked worldwide occurrences of stock symbols; this use case was inspired by our research [17]).

# Results of the Softeware-Prototyping

对12因素应用程序[18]等云原生方法的分析表明，为了建立可观测性，应该采取更细致和集成的观点来集成和统一指标、跟踪和日志这三大支柱，以便在DevOps周期的反馈信息流中实现更敏捷、更方便的分析（见图1）。在云原生计算中获得动力的两个方面令人感兴趣：


The analysis of cloud-native methodologies like the 12-factor app [18] has shown that, to build observability, one should take a more nuanced and integrated view to integrate and unify these three pillars of metrics, traces, and logs to enable more agile and convenient analytics in feedback information flow in DevOps cycles (see Figure 1). Two aspects that gained momentum in cloud-native computing are of interest:

- Recommendations on how to handle log forwarding and log consolidation in cloud- native applications;
- Recommendations to apply structured logging.

由于这两个方面都深刻地指导了日志原型的实现，因此将更详细地解释它们，为读者提供必要的上下文。

Because both aspects guided the implementation of the logging prototype deeply, they will be explained in more detail providing the reader with the necessary context.

## Twelve-Factor Apps

12因素应用程序是一种构建软件即服务应用程序的方法[18]，该应用程序特别关注应用程序随着时间的推移有机增长的动态，在代码库上合作的开发人员之间的协作障碍，以及避免软件侵蚀的成本。就其核心而言，应遵循12条规则（因素），以开发运行良好且可进化的分布式应用程序。这种方法与微服务架构方法[3,19-23]和Kubernetes[24]等云原生运营环境非常协调，这就是为什么12因素方法变得越来越流行。顺便说一句，12因素方法不包含任何明确提及可观测性的因素，当然不在度量、跟踪和日志记录三合一中。然而，因素XI建议如何处理日志记录：

The 12-factor app is a method [18] for building software-as-a-service applications that pay special attention to the dynamics of organic growth of an application over time, the dy- namics of collaboration between developers working together on a codebase, and avoiding the cost of software erosion. At its core, 12 rules (factors) should be followed to develop well- operational and evolutionarily developable distributed applications. This methodology harmonizes very well with microservice architecture approaches [3,19–23] and cloud-native operating environments like Kubernetes [24], which is why the 12-factor methodology is becoming increasingly popular. Incidentally, the 12-factor methodology does not contain any factor explicitly referring to observability, certainly not in the triad of metrics, tracing and logging. However, factor XI recommends how to handle logging:

> 日志是按时间排序并从所有运行进程和支持服务的输出流中汇总的聚合事件流。日志通常是一种文本格式，每行一个事件。[...]
> 
> Logs are the stream of aggregated events sorted by time and summarized from the output streams of all running processes and supporting services. Logs are typically a text format with one event per line. [...]
>
> 十二因素应用程序从不关心路由或存储其输出流。它不应该尝试写入或管理日志文件。相反，每个正在运行的进程都会将其事件流写入stdout。[...]在分期或生产部署时，所有进程的流都由运行时环境捕获，与应用程序的所有其他流相结合，并路由到一个或多个目的地进行查看或长期存档。这些归档目的地对应用程序既不可见，也不可配置——它们完全从运行时环境中管理。
>
> A twelve-factor app never cares about routing or storing its output stream. It should not attempt to write to or manage log files. Instead, each running process writes its stream of events to stdout. [...] On staging or production deploys, the streams of all processes are captured by the runtime environment, combined with all other streams of the app, and routed to one or more destinations for viewing or long-term archiving. These archiving destinations are neither visible nor configurable to the app—they are managed entirely from the runtime environment.

## From Logging to Structured Logging

日志记录工具对开发人员来说非常简单，主要适用于特定于编程语言，但基本上遵循Python中说明的以下原则。

The logging instrumentation is quite simple for developers and works mainly pro- gramming language specific but basically according to the following principle illustrated in Python.

必须经常导入日志库，定义所谓的日志级别，如DEBUG、INFO、WARNING、ERROR、FATAL等。当应用程序运行时，日志级别通常通过环境变量（例如INFO）设置。然后将此级别以上的所有日志调用写入日志文件。

A logging library must often be imported, defining so-called log levels such as DEBUG, INFO, WARNING, ERROR, FATAL, and others. While the application is running, a log level is usually set via an environment variable, e.g., INFO. All log calls above this level are then written to a log file.

```python
import logging
logging.basicConfig(filename="example.log", level=logging.DEBUG) logging.debug("Performing␣user␣check")
user = ‘‘Nane Kratzke’’
logging.info(f‘‘User { user } tries to log in.’’) logging.warning(f‘‘User { user } not found’) logging.error(f‘‘User␣{␣user␣}␣has␣been␣banned.’’)
```

For example, line 5 would create the following entry in a log file:

```log
INFO 2022 -01 -27 16:17:58 - User Nane Kratzke tries to log in
```

在12因素应用程序中，此日志记录将被配置为将事件直接写入Stdout（控制台）。然后，运行时环境（例如，安装了FileBeat服务的Kubernetes）将日志数据路由到适当的数据库，从开发人员那里拿走他们本来必须投资日志处理的工作。这种类型的日志记录在许多编程语言中都得到了很好的支持，并且可以与ELK堆栈（或其他可观察性堆栈）很好地整合。

In a 12-factor app, this logging would be configured so that events are written directly to Stdout (console). The runtime environment (e.g., Kubernetes with FileBeat service installed) then routes the log data to the appropriate database taking work away from the developer that they would otherwise have to invest in log processing. This type of logging is well supported across many programming languages and can be consolidated excellently with the ELK stack (or other observability stacks).

日志记录（与分布式跟踪和指标收集不同）通常甚至不被开发人员视为（复杂）工具。通常，这是他们主动完成的。然而，人们可以将该仪器系统化，并将其扩展到所谓的“结构化日志记录”。同样，原则是直截了当的。一个人根本不会像那样记录文本行

Logging (unlike distributed tracing and metrics collection) is often not even perceived as (complex) instrumentation by developers. Often, it is done on their own initiative. However, one can systematize this instrumentation somewhat and extend it to so-called
“structured logging”. Again, the principle is straightforward. One simply does not log lines of text like

```log
INFO 2022 -01 -27 16:17:58 - User Nane Kratzke tries to log in
```

but, instead, the same information in a structured form, e.g., using JSON:

```log
{‘‘log level’’: ‘‘info’’, ‘‘timestamp’’: ‘‘2022-01-27 16:17:58’’, ‘‘event ’’: ‘‘Log in’’, ‘‘user’’: ‘‘Nane Kratzke’’, ‘‘result’’: ‘‘success’’}
```

在这两种情况下，文本都写入控制台。然而，在第二种情况下，使用结构化的基于文本的数据格式，更容易评估。对于典型的日志语句，如“用户Max Mustermann尝试登录”，必须首先分析文本以确定用户。这种文本解析在大规模上成本高昂，如果有大量各种格式的日志数据（这是现实世界中的常见情况），也可以非常密集和复杂。

In both cases, the text is written to the console. In the second case, however, a struc- tured text-based data format is used that is easier to evaluate. In the case of a typical logging statement like “User Max Mustermann tries to log in", the text must first be analyzed to determine the user. This text parsing is costly on a large scale and can also be very computationally intensive and complex if there is plenty of log data in a variety of formats (which is the common case in the real world).

然而，在结构化日志记录的情况下，这些信息可以很容易地从JavaScript对象符号（JSON）数据字段“用户”中提取。特别是，通过结构化日志记录，更复杂的评估变得更加容易。然而，解释并没有变得复杂得多，特别是因为有用于结构化日志记录的日志库。本研究的日志原型log12中的日志如下所示：

However, in the case of structured logging, this information can be easily extracted from the JavaScript Object Notation (JSON) data field “user". In particular, more complex evaluations become much easier with structured logging as a result. However, the instru- mentation does not become significantly more complex, especially since there are logging libraries for structured logging. The logging looks in the logging prototype log12 of this study like this:

```python
import log12
[...]
log12.error(‘‘Log in‘‘, user=user, result="Not␣found’’,␣reason="Banned’’)
```

由此产生的日志文件对管理员和开发人员来说仍然是可读的（即使有点笨拙），但可以通过ElasticSearch等数据库更好地处理和分析。定量指标也可以以这种方式记录。因此，结构化日志记录也可用于记录定量指标：

```python
import log12
[...]
log12.info(‘‘Open requests’’, requests=len(requests))
```

```log
{ ‘‘event’’: ‘‘Open requests",␣‘‘requests": 42 }
```

此外，这种结构化的日志记录方法也可用于创建跟踪。在分布式跟踪系统中，为通过分布式系统的每笔事务创建一个跟踪标识符（ID）。各个步骤是所谓的跨度。这些还被分配了一个标识符（span ID）。然后将跨度ID链接到跟踪ID，并测量和记录运行时。通过这种方式，可以沿着所涉及的组件跟踪分布式事务的时间过程，例如，可以确定单个处理步骤的持续时间。

Furthermore, this structured logging approach can also be used to create tracings. In distributed tracing systems, a trace identifier (ID) is created for each transaction that passes through a distributed system. The individual steps are so-called spans. These are also assigned an identifier (span ID). The span ID is then linked to the trace ID, and the runtime is measured and logged. In this way, the time course of distributed transactions can be tracked along the components involved, and, for example, the duration of individual processing steps can be determined.

## Resulting and simplified Logging Architecture

因此，应用了仅将日志打印到标准输出（stdout）和以结构化和基于文本的数据格式登录的两项原则。因此，由此产生的可观测性系统复杂性从图2降低到图4，因为所有系统组件都可以以相同的风格收集日志、度量和跟踪信息，这些信息可以从提供的日志转发器（现有技术）无缝路由到中央分析数据库的操作平台。

Thus, the two principles to print logs simply to standard outout (stdout) and to log in a structured and text-based data format are applied consequently. The resulting observability system complexity thus reduces from Figure 2 to Figure 4 because all system components can collect log, metric, and trace information in the same style that can be routed seamlessly from an operation platform provided log forwarder (already existing technology) to a central analytical database.

Figure 4. An observability system consistently based on structured logging with significantly re- duced complexity.

## Study Outcome: Unified Instrumentation via a Structured Logging Library (Prototype)

本文将在下面简要解释使用出现的日志原型捕获事件、指标和跟踪的方法。原型库log12是在Python 3中开发的，但可以类似地在其他编程语言中实现。

This paper will briefly explain below the way to capture events, metrics, and traces using the logging prototype that emerged. The prototype library log12 was developed in Python 3 but can be implemented in other programming languages analogously.

Log12将自动为每个事件创建额外的键值属性，如唯一标识符（用于将子事件与分布式跟踪场景中的父事件甚至远程事件联系起来）以及可用于测量事件运行时间的开始和完成时间戳（尽管从分布式跟踪库中已知，但在日志库中并不常见）。解释了：

log12 will create automatically for each event additional key–value attributes like a unique identifier (that is used to relate child events to parent events and even remote events in distributed tracing scenarios) and start and completion timestamps that can be used to measure the runtime of events (although known from distributed tracing libraries but not common for logging libraries). It is explained:

- how to create a log stream;
- how an event in a log stream is created and logged;
- how a child event can be created and assigned to a parent event (to trace and record runtimes of more complex and dependent chains of events within the same process);
- and how to make use of the distributed tracing features to trace events that pass through a chain of services in a distributed service of services system).

以下代码行创建一个名为“logstream”的日志流，该日志流被记录到stdout，请参阅清单1：

The following lines of code create a log stream with the name “logstream” that is logged to stdout, see Listing 1:

Listing 1. Creating an event log stream in log12.

```python
import log12
log = log12.logging(‘‘logstream’’,
general=‘‘value’’, tag=‘‘foo’’, service_mark=’’test‘‘
)
```

Each event and child events of this stream are assigned a set of key–value pairs:

- general=“value”
- tag=“foo”
- service_mark=“test”

这些特定于日志流的键值对可用于在ElasticSearch等分析数据库中定义选择标准，仅过滤特定服务的事件。以下代码行演示了如何创建父事件和子事件，请参阅清单2。

These log-stream-specific key–value pairs can be used to define selection criteria in an- alytical databases like ElasticSearch to filter events of a specific service only. The following lines of code demonstrate how to create a parent event and child events, see Listing 2.

Listing 2. Event logging in log12 using blocks as structure.

```python
# Log events using the with clause
with log.event(‘‘Test’’, hello=‘‘World’’) as event: event.update(test=‘‘something’’)
# adds event specific key value pairs to the~event
with event.child(‘‘Subevent 1 of Test’’) as ev: ev.update(foo=‘‘bar’’) ev.error(’’Catastrophe’’)
# Explicit call of log (here on error level)
with event.child(‘‘Subevent 2 of Test’’) as ev: ev.update(bar=‘‘foo’’)
# Implicit call of ev.info(‘‘Success’’) (at block end)
with event.child(‘‘Subevent 3 of Test’’) as ev: ev.update(bar=‘‘foo’’)
# Implicit call of ev.info(‘‘Success’’) (at block end)
```

此外，可以在没有块样式的情况下在事件流中记录事件，请参阅清单3。对于不支持在块末尾关闭资源（此处为日志流）的编程语言来说，这可能是必要的。在这种情况下，程序员负责使用.info()、.warn()、.error()日志级别关闭事件。

Furthermore, it is possible to log events in the event stream without the block style, see Listing 3. That might be necessary for programming languages that do not support closing resources (here a log stream) at the end of a block. In this case, programmers are responsible for closing events using the .info(), .warn(), .error() log levels.

Listing 3. Event logging in log12 without blocks.

```python
# To log events without with-blocks is possible as well.
ev = log.event(‘‘Another test’’, foo=‘‘bar’’) ev.update(bar=‘‘foo’’)
child = ev.child(‘‘Subevent of Another test’’, foo=‘‘bar’’) ev.info(‘‘Finished’’)
# <= However, than~you are are responsible to log events explicitly
# If parent events are logged all subsequent child events
# are assumed to have closed successfully as well
```

使用这种类型的日志记录沿着超文本传输协议（HTTP）请求转发事件也是可能的。HTTP-Headers的这种用法是分布式跟踪的常用方法。这需要两个主要功能[25]。首先，提取HTTP服务进程收到的标头信息必须是可能的。其次，必须能够在后续上游HTTP请求中注入跟踪信息（特别是发起请求的进程的跟踪ID和跨度ID）。

Using this type of logging to forward events along Hypertext Transfer Protocol (HTTP) requests is also possible. This usage of HTTP-Headers is the usual method in distributed tracing. Two main capabilities are required for this [25]. First, extracting header information received by an HTTP service process must be possible. Secondly, it must be possible to inject the tracing information in follow-up upstream HTTP requests (in particular, the trace ID and span ID of the process initiating the request).

清单4显示了log12如何在事件创建时通过提取属性和事件注入方法来支持这一点，该方法从事件中提取相关的键值对，以便它们可以沿着HTTP请求作为标头信息传递。

Listing 4 shows how log12 supports this with an extract attribute at event creation and an inject method of the event that extracts relevant key–value pairs from the event so that they can be passed as header information along an HTTP request.

Listing 4. Extraction and injection of tracing headers in log12.

```python
import log12
import requests # To generate HTTP requests
from flask import request # To demonstrate Header~extraction
with log.event(‘‘Distributed tracing ‘‘,
extract=request.headers
) as ev:
# Here is how to pass tracing information along remote calls
with ev.child(‘‘Task 1’’) as event: response = requests.get(
‘‘https://qr.mylab.th-luebeck.dev/route?url=https://google. com’’,
headers=event.inject()
)
event.update(length=len(response.text), status=response.
status_code)
```

# 5. Evaluation of the Logging Prototype

该研究在定义的用例中评估了软件原型，以确定其是否适合捕获分布式系统中的定性事件、定量指标和跟踪。该评估还对压力测试意义上的大容量事件流进行了长期记录：

The study evaluated the software prototype in the defined use cases to determine its suitability for capturing qualitative events, quantitative metrics, and traces in distributed systems. The evaluation also performed long-term recordings of high-volume event streams in the sense of stress tests:

- The study designed the use cases 1 and 2 mainly to evaluate the instrumentation of qualitative system events;
- Use case 3 was primarily used to capture quantitative metrics that often occur in IT infrastructures or platforms and are essential to multi-level observability;
- Use case 4 was used to monitor systems that were intentionally not under the direct control of the researchers and, therefore, could not be instrumented directly. Further- more, the use case was intended to provide insight into both long-term detection and the detection of high-volume event streams.

## 5.1. Evaluation of Use Cases 1 and 2 (Event-Focused Observation of a Distributed Service)

用例1和2：Codepad是一个在线编码工具，用于在在线和离线教学场景中快速共享短代码片段。它在冠状病毒大流行关闭期间引入，主要在计算机科学第一或第二学期学生的在线教育环境中共享短代码片段。同时，该工具也用于在场讲座和实验室。欢迎读者在https://codepad.th-luebeck.dev（于2022年9月20日访问）上试用该工具。本研究在其行动研究方法的步骤1、2、3和4中使用Codepad工具作为仪器用例（见图3），根据第4.4节评估定性系统事件的仪器。图5显示了左侧的Web-UI，右侧显示了生成的仪表板。在转移步骤中（行动研究方法的步骤12、13、14和15，见图3），使用相同的产品来评估分布式跟踪仪器（本报告未详细介绍）。

Use Cases 1 and 2: Codepad is an online coding tool to share quickly short code snippets in online and offline teaching scenarios. It has been introduced during the Corona Pandemic shutdowns to share short code snippets mainly in online educational settings for 1st or 2nd semester computer science students. Meanwhile, the tool is used in presence lectures and labs as well. The reader is welcome to try out the tool at https://codepad. th-luebeck.dev (accessed on 20 September 2022). This study used the Codepad tool in its steps 1, 2, 3, and 4 of its action research methodology as an instrumentation use case (see Figure 3) to evaluate the instrumentation of qualitative system events according to Section 4.4. Figure 5 shows the Web-UI on the left and the resulting dashboard on the right. In a transfer step (steps 12, 13, 14, and 15 of the action research methodology, see Figure 3), the same product was used to evaluate distributed tracing instrumentation (not covered in detail by this report).

Figure 5. Use Cases 1 and 2: Codepad is an online coding tool to share quickly short code snippets in online and offline teaching scenarios—on the left, the Web-UI; on the right, the Kibana Dashboard used for observability in this study. Codepad was used as an instrumentation object of investigation.

## 5.2. Evaluation of Use Case 3 (Metrics-Focused Observation of Infrastructure)

用例3（研究方法的步骤5、6、7、8；图3）观察了研究所的基础设施，即所谓的myLab基础设施。myLab（可在线获取：https://mylab.th-luebeck.de）（于2022年9月20日访问）是一个虚拟实验室，学生和教职员工可以使用它来开发和托管网络应用程序。选择此用例是为了证明可以使用与用例1相同的方法长期主要收集基于指标的数据。一个豆荚主要跟踪不同大学课程的70多个学生网络项目部署的各种不同工作量的资源消耗情况。为了观察这种资源消耗，豆荚只是定期运行

The Use Case 3 (steps 5, 6, 7, 8 of research methodology; Figure 3) observed an institute’s infrastructure, the so-called myLab infrastructure. myLab (Available online: https://mylab.th-luebeck.de) (accessed on 20 September 2022) is a virtual laboratory that can be used by students and faculty staff to develop and host web applications. This use case was chosen to demonstrate that it is possible to collect primarily metrics based data over a long term using the same approach as in Use Case 1. A pod tracked mainly the resource consumption of various differing workloads deployed by more than 70 student web projects of different university courses. To observe this resource consumption, the pod simply run periodically

- kubectl top nodes;
- kubectl top pods –all-namespaces

对抗集群。该观察窗格解析了两个shell命令的输出，并打印了第4.4节中介绍的结构化日志记录方法中的解析结果。图6显示了用于演示目的的Kibana仪表板。

against the cluster. This observation pod parsed the output of both shell commands and printed the parsed results in the structured logging approach presented in Section 4.4. Figure 6 shows the resulting Kibana dashboard for demonstration purposes.

Figure 6. Use Case 3: The dashboard of the Kubernetes infrastructure under observation (myLab).

## 5.3. Evaluation of Use Case 4 (Long-Term and High-Volume Observation)

用例4（研究方法的步骤9、10、11；图3）离开了我们自己的生态系统，并观察了公共Twitter Event流作为对外部系统进行大量和长期观察的类型代表。因此，一个故意不受研究调查人员直接行政控制的系统。用例4被设计为两阶段研究。

The Use Case 4 (steps 9, 10, 11 of research methodology; Figure 3) left our own ecosystem and observed the public Twitter Event stream as a type representative for a high-volume and long-term observation of an external system. Thus, a system that was intentionally not under the direct administrative control of the study investigators. The Use Case 4 was designed as a two-phase study.

### 5.3.1. Screening Phase of Use Case 4

第一个筛选阶段旨在获得记录大批量事件流的经验，并为结构化日志库原型提供必要的功能和性能优化。筛选阶段旨在将完整和具有代表性的Twitter流量筛选为一种“地面真相”。我们对与一般推特“地面噪音”相关的语言和股票符号的分布感兴趣。这个筛选阶段从2022年1月20日持续到2022年1月30日，并确定了最常用的股票符号（见图7）。

The first screening phase was designed to gain experiences in logging high volume event streams and to provide necessary features and performance optimizations to the structured logging library prototype. The screening phase was designed to screen the complete and representative Twitter traffic as a kind of “ground truth”. We were interested in the distribution of languages and stock symbols in relation to the general Twitter “back- ground noise”. This screening phase lasted from 20 January 2022 to 30 January 2022 and identified most used stock symbols (see Figure 7).

Figure 7. Recorded events (screening phase of use case 4).

### 5.3.2. Long-Term Evaluation Phase of Use Case 4

然后进行长期记录，作为第二个长期评估阶段，并用于跟踪和记录筛选阶段确定的最常用股票符号。这个评估阶段从2022年2月持续到2022年8月中旬。在这个评估阶段，由于作者研究所停电，只发生了一次基础设施停机。然而，此停机不是由于或与呈现的统一日志堆栈有关（见图8）。

A long-term recording was then done as a second long-term evaluation phase and was used to track and record the most frequent used stock symbols identified in the screening phase. This evaluation phase lasted from February 2022 until the middle of August 2022. In this evaluation phase, just one infrastructure downtime occurred due to a shutdown of electricity of the author’s institute. However, this downtime was not due to or related to the presented unified logging stack (see Figure 8).

Figure 8. Recorded events (long-term evaluation phase of use case 4).

### 5.3.3. The Instrumentation Approach Applied in Both Phases

录音使用以下源代码完成，请参阅清单5，编译成Docker容器，该容器已在已登录到用例1、2和3中的Kubernetes集群上执行。FileBeat被用作后台ElasticSearch数据库的日志转发组件。使用Kibana对生成的事件日志进行了分析和可视化。Kibana还用于以CSV文件的形式收集筛选和评估阶段的数据。图7、9和10是根据这些数据汇编的。这一设置完全遵循了图4中显示的统一和简化的日志记录架构。

The recording was done using the following source code, see Listing 5, compiled into a Docker container, that has been executed on a Kubernetes cluster that has been logged in Use Cases 1, 2, and 3. FileBeat was used as a log forwarding component to a background ElasticSearch database. The resulting event log has been analyzed and visualized using Kibana. Kibana was used as well to collect the data in form of CSV-Files for the screening and the evaluation phase. Figures 7, 9 and 10 have been compiled from that data. This setting followed the unified and simplified logging architecture presented in Figure 4 exactly.

Listing 5. The used logging program to record Twitter stock symbols from the public Twitter Stream API.

```python
import log12 , tweepy , os
KEY = os.environ.get(‘‘CONSUMER_KEY’’)
SECRET = os.environ.get(‘‘CONSUMER_SECRET’’)
TOKEN = os.environ.get(‘‘ACCESS_TOKEN’’)
TOKEN_SECRET = os.environ.get(‘‘ACCESS_TOKEN_SECRET’’)
LANGUAGES = [l.strip() for l in os.environ.get(‘‘LANGUAGES’’, ‘‘’’).split (‘‘,’’)]
TRACK = [t.strip() for t in os.environ.get(‘‘TRACKS’’).split(‘‘,’’)] log = log12.logging(‘‘twitter stream’’)
class Twista(tweepy.Stream):
def on_status(self, status):
with log.event(‘‘tweet’’, tweet_id=status.id_str,
user_id=status.user.id_str , lang=status.lang ) as event:
kind = ‘‘status’’
kind = ’’reply’’ if status._json[’in_reply_to_status_id’] else kind
kind = ‘‘retweet’’ if ’retweeted_status’ in status._json else kind
kind = ‘‘quote’’ if ’quoted_status’ in status._json else kind event.update(lang=status.lang, kind=kind, message=status.text
)
with event.child(’user’) as usr:
name = status.user.name if status.user.name else ‘‘
unknown’’
usr.update(lang=status.lang, id=status.user.id_str,
name=name, screen_name=f‘‘@{status.user.screen_name}’’, message=status.text,
kind=kind
)
in status.entities[’hashtags’]:
for tag
with event.child(’hashtag’) as hashtag:
hashtag.update(lang=status.lang, tag=f‘‘#{tag[’text’].lower()}’’, message=status.text,
kind=kind
)
for sym in status.entities[’symbols’]: with event.child(’symbol’) as symbol:
symbol.update(lang=status.lang, symbol=f‘‘${sym[’text’].upper()}’’, message=status.text,
kind=kind
) symbol.update(screen_name=f‘‘@{status.user.
screen_name}’’)
for user_mention in status.entities[’user_mentions’]: with event.child(’mention’) as mention:
mention.update(lang=status.lang, screen_name=f‘‘@{user_mention[’screen_name’]}’’, message=status.text,
kind=kind
)
record = Twista(KEY, SECRET, TOKEN, TOKEN_SECRET) if LANGUAGES:
record.filter(track=TRACK , languages=LANGUAGES) else:
record.filter(track=TRACK)
```

### 5.3.4. Observed and Recorded Twitter Behaviour in Both Phases of Use Case 4

根据图7和图8，筛选阶段每观察到的第100个事件就有一个股票符号。这只是推特上的“真相”。如果一个人在没有任何过滤器的情况下观察公共推特流，这就是你得到的。因此，第二个评估阶段记录了Twitter流非常具体的“过滤器泡沫”。读者应该知道，以下数据是明显的偏见，而不是具有代表性的Twitter事件流；它显然是一个以股市为重点的子集，或者更准确地说，是一个以加密货币为重点的子集，因为Twitter上的几乎所有股票符号都与加密货币有关。

According to Figures 7 and 8, just every 100th observed event in the screening phase was a stock symbol. That is simply the “ground-truth” on Twitter. If one is observing the public Twitter stream without any filter, that is what you get. Thus, the second evaluation phase recorded a very specific “filter bubble” of the Twitter stream. The reader should be aware that the data presented in the following is a clear bias and not a representative Twitter event stream; it is clearly a stock market focused subset or, to be even more precise, a cryptocurrency focused subset, because almost all stock symbols on Twitter are related to cryptocurrencies.

可以使用记录的数据可视化结果的效果。图9显示了筛选阶段（未经过滤的地面真相）和评估阶段（激活符号过滤器）的语言分布差异。然而，在筛选阶段，英语（en）、西班牙语（es）、葡萄牙语（pt）和土耳其语（tr）占所有流量的3/4以上；在评估阶段，几乎所有录制的推文都是英文的。因此，在推特上，最与股票符号相关的语言显然是英语。

It is possible to visualize the resulting effects using the recorded data. Figure 9 shows the difference in language distributions of the screening phase (unfiltered ground-truth) and the evaluation phase (activated symbol filter). However, in the screening phase, English (en), Spanish (es), Portugese (pt), and Turkish (tr) are responsible for more than 3/4 of all traffic; in the evaluation phase, almost all recorded Tweets are in English. Thus, on Twitter, the most stock symbol related language is clearly English.

Figure 9. Observed languages (screening and evaluation phase of Use Case 4).

虽然加密货币日志记录主要用作日志库原型技术评估的用例，但可以获得一些有趣的见解。例如，尽管比特币（BTC）可能是最突出的加密货币，但它目前还不是Twitter上最常用的股票符号。Twitter上最突出的股票符号是：

Although the cryptocurrency logging was used mainly as a use case for technical evaluation purposes of the logging library prototype, some interesting insights could be gained. For example, although Bitcoin (BTC) is likely the most prominent cryptocurrency, it is by far not the most frequently used stock symbol on Twitter. The most prominent stock symbols on Twitter are:

- ETH: Ethereum cryptocurrency;
- SOL: Solana cryptocurrency;
- BTC: Bitcoin cryptocurrency;
- LUNA: Terra Luna cryptocurrency (replaced by a new version after the crash in
May 2022);
- BNB: Binance Coin cryptocurrency.
Furthermore, we can see interesting details in trends (see Figure 10).
- The ETH usage on Twitter seems to reduce throughout our observed period;
- The SOL usage is, on the contrary, increasing, although we observed a sharp decline
in July.
- The LUNA usage has a clear peak that correlates with the LUNA cryptocurrency crash
in the middle of May 2022 (this crash was heavily reflected in the investor media).

Twitter的使用与加密货币股市的货币汇率无关。然而，作为值得观察的有趣指标，加密货币投资者可能会对股市符号使用模式的变化感兴趣。正如这项研究表明的那样，使用结构化测井方法可以轻松跟踪这些变化。当然，这也可以转移到其他社交媒体流媒体或一般事件流媒体用例，如物联网（物联网）。

The Twitter usage was not correlated with the currency rates in cryptocurrency stock markets. However, changes in usage patterns of stock market symbols might be of interest for cryptocurrency investors as interesting indicators to observe. As this study shows, these changes can be easily tracked using structured logging approaches. Of course, this can be transferred to other social media streaming or general event streaming use cases like IoT (Internet of Things) as well.

Figure 10. Recorded symbols per day (evaluation phase of Use Case 4).

# 6. Discussion

在使用基于FileBeat/ElasticSearch的可观察性堆栈的几个用例上成功评估了这种统一和结构化的可观测性风格。然而，其他可以以JSON格式转发和解析结构化文本的可观察性堆栈可能会显示相同的结果。评估包括对大量评估用例进行超过六个月的长期测试。

This style of a unified and structured observability was successfully evaluated on several use cases that made usage of a FileBeat/ElasticSearch-based observability stack. However, other observability stacks that can forward and parse structured text in a JSON- format will likely show the same results. The evaluation included a long-term test over more than six months for a high-volume evaluation use-case.

- 一方面，可以证明，这种类型的日志记录可以很容易地用于执行经典的指标集合。为此，在几个Kubernetes集群中成功收集和评估了BlackBox指标，如基础设施（节点）的CPU、内存和存储，以及“有效负载”（pod）（见图6）。
- 其次，对大量用例进行了深入调查和分析。在这里，公共推特流上的所有英语推文都被记录下来了。在一周内，每小时记录约100万个事件，并使用日志转发器FileBeat转发到ElasticSearch数据库。大多数系统产生的事件会少得多（见图7）。
- 此外，原型日志库log12同时用于几个外部系统，包括基于Web的开发环境、二维码服务和电子学习系统，以记录学习内容的访问频率，并研究学生的学习行为。

- On the one hand, it could be proven that such a type of logging can easily be used to perform classic metrics collections. For this purpose, BlackBox metrics such as CPU, memory, and storage for the infrastructure (nodes) but also the “payload” (pods) were successfully collected and evaluated in several Kubernetes clusters (see Figure 6).
- Second, a high-volume use case was investigated and analyzed in-depth. Here, all English-language tweets on the public Twitter stream were logged. About 1 million events per hour were logged over a week and forwarded to an ElasticSearch database using the log forwarder FileBeat. Most systems will generate far fewer events (see Figure 7).
- In addition, the prototype logging library log12 is meanwhile used in several in- ternal systems, including web-based development environments, QR code services, and e-learning systems, to record access frequencies to learning content, and to study learning behaviours of students.

## 6.1. Lessons Learned

所有用例都表明，结构化日志记录易于处理，并与现有的可观测性堆栈很好地协调（特别是Kubernetes、Filebeat、ElasticSearch、Kibana）。然而，应该考虑一些方面：

All use cases have shown that structured logging is easy to instrument and harmonizes well with existing observability stacks (esp. Kubernetes, Filebeat, ElasticSearch, Kibana). However, some aspects should be considered:

1. 应用结构化日志记录至关重要，因为这可用于以相同风格记录事件、指标和跟踪。It is essential to apply structured logging, since this can be used to log events, metrics, and traces in the same style.
2. 通常，只记录容易出错的情况。但是，如果您想在符合DevOps的可观察性意义上行事，您还应该记录正常（完全正常）的行为。DevOps工程师可以从普通用户如何在标准情况下使用系统中获得许多见解。因此，日志级别应设置为INFO，而不是警告、错误或以下。Very often, only error-prone situations are logged. However, if you want to act in the sense of DevOps-compliant observability, you should also log normal—completely regular—behaviour. DevOps engineers can gain many insights from how normal users use systems in standard situations. Thus, the log level should be set to INFO, and not WARNING, ERROR, or below.
3. 云原生系统组件应依赖于运行时环境的日志转发和日志代理。永远不要自己实现这一点。您将使用双重逻辑，最终出现复杂且可能不兼容的日志聚合系统。Cloud-native system components should rely on the log forwarding and log aggrega- tion of the runtime environment. Never implement this on your own. You will double logic and end up with complex and may be incompatible log aggregation systems.
4. 为了简化工程师的分析，应该将父事件的键值对推到子事件。这种日志记录方法简化了集中日志分析解决方案中的分析——它只是减少了在JSON文档存储中可能难以推断的事件上下文的需求。然而，这伴随着更广泛的原木存储成本。To simplify analysis for engineers, one should push key–value pairs of parent events down to child events. This logging approach simplifies analysis in centralized log analysis solutions—it simply reduces the need to derive event contexts that might be difficult to deduce in JSON document stores. However, this comes with the cost of more extensive log storage.
5. 不要收集汇总的指标数据。聚合（平均值、中位数、百分位数、标准差、和、计数等）可以在分析数据库中更方便地完成。该仪器应专注于以点对点风格记录指标数据。根据我们的开发人员经验，开发人员很高兴被授权只记录这些简单的指标，特别是在统计学方面没有太多背景知识时。Do not collect aggregated metrics data. The aggregation (mean, median, percentile, standard deviations, sum, count, and more) can be done much more conveniently in the analytical database. The instrumentation should focus on recording metrics data in a point-on-time style. According to our developer experience, developers are happy to be authorized to log only such simple metrics, especially when there is not much background knowledge in statistics.

## 6.2. Threats of Validity and to Be Considered Limitations of the Study Design

行动研究容易得出不正确或不可概括的结论。逻辑上所述，在考虑的用例中，其重要性始终最高。为了最大限度地减少此类风险，读者应考虑以下对有效性的威胁（见[26,27]）：

Action research is prone to drawing incorrect or non-generalizable conclusions. Logi- cally, the significance is consistently the highest within the considered use cases. In order to minimize such kind of risks, the reader should consider the following threats on validity (see [26,27]):

- 构造有效性是指所采用的措施是否适当地反映了它们所代表的构造的问题。Construct validity refers to the question, whether the employed measures appropri- ately reflect the constructs they represent.
- 内部有效性是指观察到的关系是否是由于因果关系的问题。因此，它仅与确定因果关系的案例研究相关[28]，而所提出的软件原型研究并非如此。Internal validity refers to the question of whether observed relationships are due to a cause–effect relationship. Thus, it is only relevant for case studies in which a cause–effect is to be determined [28], which is not the case for the presented software prototype study.
- 外部有效性是指案例研究的发现是否可以推广的问题。External validity refers to the question of whether the findings of the case study can be generalized.
- 可靠性或实验有效性是指是否可以以相同的结果重复研究的问题。Reliability or experimental validity refers to the question of whether the study can be repeated with the same results.

因此，报告了对结构和外部有效性的考虑，以及对可重复性的考虑。

Therefore, considerations on construct and external validity are reported as well as considerations on repeatability.

### 6.2.1. Considerations on Construct Validity

为了得出可概括的结论，本研究以故意考虑不同类别的遥测数据（日志、度量、跟踪）的方式定义了用例。应该指出的是，研究设计主要考虑日志和指标，但仅略有跟踪。然而，痕迹并没有完全被忽视，但分析不那么密集。然而，研究设计故意涵盖了日志、痕迹和指标，看看它们是否可以使用一致的仪器方法进行记录。这种仪器方法旨在生成一个结构化的时间序列数据集，该数据集可以使用现有的可观测性工具堆栈进行整合和分析。

In order to draw generalizable conclusions, this study defined use cases in such a way that intentionally different classes of telemetry data (logs, metrics, traces) were considered. It should be noted that the study design primarily considered logs and metrics but traces only marginally. Traces were not wholly neglected, however, but were analyzed less intensively. However, the study design deliberately covered logs, traces, and metrics to see if they could be recorded using a consistent instrumentation approach. This instrumentation approach was designed to generate a structured time-series dataset that can be consolidated and analyzed using existing observability tool stacks.

该研究在四个用例中评估了由此产生的软件原型，以确定其是否适合在分布式系统中捕获定性事件、定量指标和跟踪。该评估还对压力测试意义上的大容量事件流进行了长期记录。虽然此设置旨在涵盖广泛的网络服务和网络应用程序领域，但在此设置的范围之外，不应将研究结果用于得出任何结论。例如，读者不应该对通常需要考虑微秒延迟的硬实时场景的适用性做出任何假设。

The study evaluated the resulted software prototype in four use cases to determine its suitability for capturing qualitative events, quantitative metrics, and traces in distributed systems. The evaluation also performed long-term recordings of high-volume event streams in the sense of stress tests. Although this setting was constructed to cover a broad range of web-service and web-application domains, outside the scope of this setting, the study outcomes should not be taken to draw any conclusions. For instance, the reader should not make any assumptions on the applicability in hard real-time scenarios where microsecond latencies often have to be considered.

### 6.2.2. Considerations on External Validity

这项研究通过在仪器端为Python 3编程语言创建软件原型并在分析端使用ELK堆栈来证明概念。尽管如此，还需要进一步努力将结果转移到其他编程语言和可观察性堆栈。由于可以假设这两个基本原则（结构化日志记录和时间序列数据库和分析）都是已知和掌握的，因此读者可以预计这里不会遇到技术性的重大困难。

This study did the proof of concept by creating a software prototype for the Python 3 programming language on the instrumentation side and using the ELK stack on the analysis side. Nevertheless, further efforts are needed for transferring the results to other programming languages and observability stacks. Since both basic principles (structured logging and time-series databases and analyses) can be assumed to be known and mastered, the reader can expect no significant difficulties of a technical nature here.

长期收购使用大量用例进行，以涵盖某些应力测试方面。然而，读者必须意识到，筛选阶段在用例4中产生的数据量明显高于评估阶段。因此，要使用本研究的压力测试数据，应该查看用例4筛选阶段的事件量。在这里，每分钟记录约一万个事件超过一周，给人留下了拟议方法表现的印象。研究数据显示，饱和度限制应该远远超过每分钟一万个事件。然而，研究设计并没有将系统推向事件记录饱和度极限。

The long-term acquisition was performed with a high-volume use case to cover certain stress test aspects. However, the reader must be aware that the screening phase generated significantly higher data volumes in Use Case 4 than the evaluation phase. Therefore, to use stress test data from this study, one should look at the event volume of the screening phase of Use Case 4. Here, about ten thousand events per minute were logged for more than a week giving an impression of the performance of the proposed approach. The study data show that the saturation limit should be far beyond these ten thousand events per minute. However, the study design did not push the system to its event recording saturation limits.

此外，本研究不应用于得出任何与加密货币相关的结论。虽然使用案例4中的一些有趣的方面可能对加密货币交易指标的生成感兴趣，但尚未对股票价格与Twitter上股票符号使用频率之间的相关性进行详细分析。

Furthermore, this study should not be used to derive any cryptocurrency related conclusions. Although some interesting aspects from Use Case 4 could be of interest for cryptocurrency trading indicator generation, no detailed analysis on correlations between stock prices and usage frequencies of stock symbols on Twitter have been done.

### 6.2.3. Considerations on Repeatability

为了让读者有机会复制此处介绍的用例，log12 [29]的源代码已作为开源软件提供。我们还报告了本研究中使用的应用可观测性堆栈（ELK堆栈）。

To give the reader the opportunity to reproduce the use cases presented here, source code from log12 [29] has been made available as open source software. We also reported on the applied observability stack that has been used in this study (ELK-Stack).

# 7. Related Work

最近发布的异常检测和根本原因分析调查也讨论了指标、痕迹和日志的三大支柱[30]。然而，自动日志解析的工具和基准[31]或像[32]这样的有趣出版物的可观察性通常会降低，专注于日志分析的进展和挑战。像[33,34]这样的出版物报告了关于开发人员如何记录或如何改进一般日志记录的实证研究。像[35,36]这样的研究更侧重于日志中的异常检测。然而，与此同时，所有与日志相关的研究都有点过时。此外，在IT安全和异常检测的背景下，日志记录通常与日志文件分析有关[37]。

The three pillars of metrics, traces, and logs have also been tackled in a recently published survey on anomaly detection and root cause analysis [30]. However, very often observability is reduced on tools and benchmarks for automated log parsing [31] or interesting publications like [32] focus on advances and challenges in log analysis. Publications like [33,34] report on empirical studies regarding how developers log or how to improve logging in general. Studies like [35,36] focus more on anomaly detection in logs. However, all log-related studies are meanwhile a bit outdated. In addition, very often logging is related to log file analysis in the context of IT security and anomaly detection only [37].

最近的研究从更全面的角度来看待可观测性。参考文献[38]明确关注分布式系统的可观测性和监测性。参考文献[39]将微服务集中在这种可观测性背景下。参考文献[40]重点关注多级可观测性的需求，特别是在编排方法中。此外，参考文献[41]考虑了可扩展的可观测性数据管理。[42]提供了关于分布式边缘和基于容器的微服务的可观测性的最新有趣概述。这项调查提供了以微服务为重点的管理和统一的可观测性服务列表（Dynatrace、Datadog、New Relic、Sumo Logic、Solar Winds、Honeycomb）。本研究的预先发送的研究原型朝同一方向发展，但试图使用更轻量级和统一的方法主要在仪器方面解决这个问题。因此，解决这个问题的客户端显然更难利用，这就是为什么该行业最好在托管服务方面解决这个问题。

More recent studies look at observability from a more all-encompassing point of view. Ref. [38] focus explicitly on the observability and monitoring of distributed sys- tems. Ref. [39] focuses microservices in this observability context. Ref. [40] focuses the need of multi-level observability especially in orchestration approaches. In addition, Ref. [41] considers scalable observability data management. An interesting and recent overview on observability of distributed edge and container-based microservices is provided by [42]. This survey provides a list of microservice-focused managed and unified observability services (Dynatrace, Datadog, New Relic, Sumo Logic, Solar Winds, Honeycomb). The pre- sented research prototype of this study heads into the same direction but tries to pursue the problem primarily on the instrumenting side using a more lightweight and unified approach. Thus, to address the client-side of the problem is obviously harder economi- cal exploitable, which is why the industry might address the problem preferably on the managed service side.

在日志、指标和分布式跟踪中，分布式跟踪仍然被最详细地考虑。特别是，这里应该提到围绕Dapper[25]（最初由谷歌运营的大规模分布式系统跟踪基础设施）的论文，这对该领域产生了重大影响。[43]提出了一种没有仪器分布式跟踪所需的黑匣子方法。它主要通过统计手段报告大规模互联网服务的端到端性能分析。然而，这些黑匣子方法的表达能力非常有限，因为仅由于其可观察到的网络行为，操作必须记录大型数据集，才能沿着分布式系统的组件进行交易。与此同时，它已成为接受白盒仪器努力的住所，以便能够确定和统计评估精确的交易流程。在此背景下，参考文献[44]比较和评估现有的开放跟踪工具。参考文献[45]概述了如何跟踪基于组件的分布式系统。此外，参考文献[46]专注于分布式跟踪的自动分析以及相应的挑战和研究方向。然而，这项研究认为追踪只是可观测性的三个方面之一，因此遵循更广泛的方法。最重要的是，这项研究的重点是可观测性的仪器方面，而不是数据库和时间序列分析方面。

Of logs, metrics, and distributed traces, distributed tracing is still considered in the most detail. In particular, the papers around Dapper [25] (a large-scale distributed systems tracing infrastructure initially operated by Google) should be mentioned here, which had a significant impact on this field. A black box approach without instrumenting needs for distributed tracing is presented by [43]. It reports on end-to-end performance analysis of large-scale internet services mainly by statistical means. However, these black-box approaches are pretty limited in their expressiveness since operations must record large data sets to derive transactions along distributed systems’ components simply due to their observable network behaviour. In the meantime, it has become an abode to accept the effort of white-box instrumentation to be able to determine and statistically evaluate precise transaction processes. In this context, Ref. [44] compares and evaluates existing open tracing tools. Ref. [45] provides an overview of how to trace distributed component-based systems. In addition, Ref. [46] focuses on automated analysis of distributed tracing and corresponding challenges and research directions. This study, however, has seen tracing as only one of three aspects of observability and therefore follows a broader approach. Most importantly, this study has placed its focus on the instrumentation side of observability and less on the database and time series analysis side.

## 7.1. Existing Instrumenting Libraries and Observability Solutions

虽然可观测领域的学术覆盖范围是可扩展的，但在实践中，有一套广泛的现有解决方案，特别是在时间序列分析和仪器内。完整的清单超出了本文的范围。然而，从学术论文数量不成比例到实际现有解决方案的数量，人们很快就认识到了该主题的实际相关性。表1包含通常用于遥测数据整合的前系统数据库产品列表，以便在不声称完整性的情况下为读者提供概述。这项研究使用ElasticSearch作为分析数据库。

Although the academic coverage of the observability field is expandable, in practice, there is an extensive set of existing solutions, especially for time series analysis and in- strumentation. A complete listing is beyond the scope of this paper. However, from the disproportion of the number of academic papers to the number of real existing solutions, one quickly recognizes the practical relevance of the topic. Table 1 contains a list of ex- isting database products often used for telemetry data consolidation to give the reader an overview without claiming completeness. This study used ElasticSearch as an analyti- cal database.

Table 1. Often seen databases for telemetry data consolidation. Products used in this study are marked bold ⊗, without claiming completeness.

Table 2 lists several frequently used forwarding solutions that developers can use to forward data from the point of capture to the databases listed in Table 1. In the context of this study, FileBeat was used as a log forwarding solution. It could be proved that this solution is also capable of forwarding traces and metrics if applied in a structured logging setting.

Table 2. Often seen forwarding solutions for log consolidation. Products used in this study are marked bold ⊗, without claiming completeness.

表3无疑对不同产品和语言的仪器库进行了不完整的概述，大概是因为每种编程语言都有自己以特定库的形式进行日志记录。除非像[43]这样的“深奥方法”，否则在仪器上下文中几乎不可能避免这种语言绑定。日志库原型受到Python标准日志库的强烈影响，但也受到结构化日志的影响，但实际上并没有使用这些库。

An undoubtedly incomplete overview of instrumentation libraries for different prod- ucts and languages is given in Table 3, presumably because each programming language comes with its own form of logging in the shape of specific libraries. To avoid this language- binding is hardly possible in the instrumentation context unless one pursues “esoteric approaches” like [43]. The logging library prototype is strongly influenced by the Python standard logging library but also by structlog for structured logging but without actually using these libraries.

Table 3. Often seen instrumenting libraries. Products that inspired the research prototype are marked bold ⊗, without claiming completeness.

## 7.2. Standards

几乎没有任何可观测性标准。然而，一个值得注意的标准方法是云原生计算基金会[66]的OpenTelemetry规范[10]，该规范试图标准化仪器化方式。这种方法符合核心思想，本研究也遵循了核心思想。尽管如此，该标准仍分为日志[67]、指标[68]和跟踪[69]，这意味着可观测性的概念三和弦不受质疑。另一方面，Kubernetes的Open- Telemetry Operator [70]等方法允许将Java、Node.js和Python的自动仪器库注入Kubernetes操作的应用程序，这是本研究目前尚未涉及的功能。然而，所谓的服务网格[71,72]也使用自动仪器。这里的一个开发标准是所谓的服务网格接口（SMI）[73]。

There are hardly any observability standards. However, a noteworthy standardiza- tion approach is the OpenTelemetry Specification [10] of the Cloud Native Computing Foundation [66], which tries to standardize the way of instrumentation. This approach corresponds to the core idea, which this study also follows. Nevertheless, the standard is still divided into Logs [67], Metrics [68] and Traces [69], which means that the conceptual triad of observability is not questioned. On the other hand, approaches like the Open- Telemetry Operator [70] for Kubernetes enable injecting auto-instrumentation libraries for Java, Node.js and Python into Kubernetes operated applications, which is a feature that is currently not addressed by the present study. However, so-called service meshes [71,72] also use auto-instrumentation. A developing standard here is the so-called Service Mesh Interface (SMI) [73].

# 8. Conclusions and Future Research Directions

云原生软件系统通常具有更分散的结构和许多可独立部署和（横向）可扩展的组件，这使得创建整体分散式系统状态的共享和整合图像变得更加复杂[74,75]。今天，可观测性通常被理解为收集和处理指标、分布式跟踪数据和日志记录的三和弦——但为什么除了历史原因？

Cloud-native software systems often have a much more decentralized structure and many independently deployable and (horizontally) scalable components, making it more complicated to create a shared and consolidated picture of the overall decentralized system state [74,75]. Today, observability is often understood as a triad of collecting and processing metrics, distributed tracing data, and logging—but why except for historical reasons?

本研究介绍了Python[29]的统一日志记录库和使用结构化日志记录方法的统一日志记录架构（见图4）。对四个用例的评估表明，每分钟数千个事件易于处理，可用于处理日志、跟踪和指标。至少，这项研究能够以直截了当的方法来记录六个月内全球股市符号的Twitter事件流，而没有任何值得注意的问题。作为副作用，可以推导出加密货币如何在Twitter上反映的一些有趣方面。这可能与这项研究无关紧要，但显示了基于日志的统一和结构化可观测性方法的总体潜力。

This study presents a unified logging library for Python [29] and a unified logging architecture (see Figure 4) that uses a structured logging approach. The evaluation of four use cases shows that several thousand events per minute are easily processable and can be used to handle logs, traces, and metrics the same. At least, this study was able with a straightforward approach to log the world-wide Twitter event stream of stock market symbols over a period of six months without any noteworthy problems. As a side effect, some interesting aspects of how crypto-currencies are reflected on Twitter could be derived. This might be of minor relevance for this study but shows the overall potential of a unified and structured logging based observability approach.

该方法依赖于易于使用的特定于编程语言的日志记录库，该库遵循结构化日志记录方法。超过六个月的长期观测结果表明，无需开发全新的工具链，即可统一当前日志、度量和跟踪的可观测性三和弦。原因是基础结构化日志记录方法的灵活性。这种灵活性是数据格式标准化的典型效果。诀窍是

The presented approach relies on an easy-to-use programming language-specific logging library that follows the structured logging approach. The long-term observation results of more than six months indicate that a unification of the current observability triad of logs, metrics, and traces is possible without the necessity to develop utterly new toolchains. The reason is the flexibility of the underlying structured logging approach. This kind of flexibility is a typical effect of data format standardization. The trick is to

- use structured logging and
- apply log forwarding to a central analytical database
- in a systematic infrastructure- or platform-provided way.

因此，进一步的研究应该集中在仪器上，而不是对数转发和整合层。如果我们使用相同的日志转发以相同的风格检测日志、跟踪和指标，我们将自动在单个真实数据源中生成相关数据，并简化分析。

Further research should therefore be concentrated on the instrumenting and less on the log forwarding and consolidation layer. If we instrument logs, traces, and metrics in the same style using the same log forwarding, we automatically generate correlatable data in a single data source of truth, and we simplify analysis.

因此，前面的可观测性道路可能有几条路径。一方面，我们应该以结构化风格标准化日志记录库，如本研究中的log12或“野生”中的OpenTelemetry项目。日志库应以不同的编程语言进行比较实现，并应生成相同的结构化日志记录数据。因此，我们必须标准化日志SDK和数据格式。两者都应设计为以结构化格式覆盖日志、指标和分布式跟踪。为了进一步模拟仪器，我们还应该考虑自动仪器方法，例如OpenTelemetry Kubernetes运算符[70]和Istio[76,77]等几个服务网格以及SMI[73]等相应标准提出的自动仪器方法。

Thus, the observability road ahead may have several paths. On the one hand, we should standardize the logging libraries in a structured style like log12 in this study or the OpenTelemetry project in the “wild”. Logging libraries should be comparably implemented in different programming languages and shall generate the same structured logging data. Thus, we have to standardize the logging SDKs and the data format. Both should be designed to cover logs, metrics, and distributed traces in a structured format. To sim- plify instrumentation further, we should additionally think about auto-instrumentation approaches, for instance, proposed by the OpenTelemetry Kubernetes Operator [70] and several Service Meshes like Istio [76,77] and corresponding standards like SMI [73].

Funding: This research received no external funding.

Data Availability Statement: The resulting research prototype of the developed structured logging library log12 can be accessed here [29]. However, the reader should be aware, that this is prototyping software in progress.

Conflicts of Interest: The author declares no conflict of interest.

# Reference

1. Kalman, R. On the general theory of control systems. IFAC Proc. Vol. 1960, 1, 491–502. https://doi.org/10.1016/S1474- 6670(17)70094-8.
2. Kalman, R.E. Mathematical Description of Linear Dynamical Systems. J. Soc. Ind. Appl. Math. Ser. A Control 1963, 1, 152–192. https://doi.org/10.1137/0301010.
3. Newman, S. Building Microservices, 1st ed.; O’Reilly Media, Inc.: Sebastopol, CA, USA, 2015.
4. Kim, G.; Humble, J.; Debois, P.; Willis, J.; Forsgren, N. The DevOps Handbook: How to Create World-Class Agility, Reliability, &
SecurityinTechnologyOrganizations;ITRevolution: Sebastopol,CA,USA,2016.
5. Davis, C. Cloud Native Patterns: Designing Change-Tolerant Software; Simon and Schuster: New York, NY, USA, 2019.
6. Kratzke, N. Cloud-native Computing: Software Engineering von Diensten und Applikationen für die Cloud; Carl Hanser Verlag GmbH
Co. KG: Munich, Germany, 2021.
7. Rochim, A.F.; Aziz, M.A.; Fauzi, A. Design Log Management System of Computer Network Devices Infrastructures Based on
ELK Stack. In Proceedings of the 2019 International Conference on Electrical Engineering and Computer Science (ICECOS),
Batam Island, Indonesia, 2–3 October 2019; pp. 338–342.
8. Lahmadi, A.; Beck, F. Powering monitoring analytics with elk stack. In Proceedings of the 9th International Conference on
Autonomous Infrastructure, Management and Security (Aims 2015), Ghent, Belgium, 22–25 June 2015.
9. APM Authors. APM: Application Performance Monitoring. 2022. Available online: https://www.elastic.co/observability/
application-performance-monitoring (accessed on 20 September 2022).
10. The OpenTelemetry Authors. The OpenTelemetry Specification. 2021. Available online: https://github.com/open-telemetry/
opentelemetry-specification/releases/tag/v1.12.0 (accessed on 20 September 2022).
11. Kratzke, N.; Quint, P.C. Understanding Cloud-native Applications after 10 Years of Cloud Computing-A Systematic Mapping
Study. J. Syst. Softw. 2017, 126, 1–16. https://doi.org/10.1016/j.jss.2017.01.001.
12. Kratzke, N. A Brief History of Cloud Application Architectures. Appl. Sci. 2018, 8, 1368. https://doi.org/10.3390/app8081368.
13. Bader, A.; Kopp, O.; Falkenthal, M. Survey and comparison of open source time series databases. In Datenbanksysteme für Business,
Technologie und Web (BTW 2017)-Workshopband; Gesellschaft für Informatik, Bonn, Germany, 2017.
14. Petersen, K.; Gencel, C.; Asghari, N.; Baca, D.; Betz, S. Action Research as a Model for Industry-Academia Collaboration in the Software Engineering Context. In Proceedings of the 2014 International Workshop on Long-Term Industrial Collaboration on
Software Engineering, WISE ’14, Vasteras, Sweden, 16 September 2014; Association for Computing Machinery: New York, NY,
USA, 2014; pp. 55–62. https://doi.org/10.1145/2647648.2647656.
15. Kratzke, N. Smart Like a Fox: How clever students trick dumb programming assignment assessment systems. In Proceedings of
the 11th International Conference on Computer Supported Education (CSEDU 2019), Heraklion, Greece, 2–4 May 2019.
16. Truyen, E.; Kratzke, N.; Van Landuyt, D.; Lagaisse, B.; Joosen, W. Managing Feature Compatibility in Kubernetes: Vendor
Comparison and Analysis. IEEE Access 2020, 8, 228420–228439. https://doi.org/10.1109/ACCESS.2020.3045768.
17. Kratzke, N. The #BTW17 Twitter Dataset-Recorded Tweets of the Federal Election Campaigns of 2017 for the 19th German
Bundestag. Data 2017, 2, 34. https://doi.org/10.3390/data2040034.
18. Wiggins, A. The Twelve-Factor App. 2017. Available online: https://12factor.net (accessed on 20 September 2022).
19. Dragoni, N.; Giallorenzo, S.; Lafuente, A.L.; Mazzara, M.; Montesi, F.; Mustafin, R.; Safina, L. Microservices: Yesterday, today, and tomorrow. In Present and Ulterior Software Engineering; Springer: Berlin/Heidelberg, Germany, 2017, pp. 195–216.
20. Taibi, D.; Lenarduzzi, V.; Pahl, C. Architectural patterns for microservices: A systematic mapping study. In Proceedings of the CLOSER 2018: The 8th International Conference on Cloud Computing and Services Science, Funchal, Portugal, 19–21 March 2018; SciTePress; Setubal, Portugal: 2018.
21. Di Francesco, P.; Lago, P.; Malavolta, I. Architecting with microservices: A systematic mapping study. J. Syst. Softw. 2019, 150, 77–97.
22. Soldani, J.; Tamburri, D.A.; Van Den Heuvel, W.J. The pains and gains of microservices: A systematic grey literature review. J. Syst. Softw. 2018, 146, 215–232.
23. Baškarada, S.; Nguyen, V.; Koronios, A. Architecting microservices: Practical opportunities and challenges. J. Comput. Inf. Syst. 2020, 60, 428–436.
24. The Kubernetes Authors. Kubernetes, 2014. Available online: https://kubernetes.io (accessed on 20 September 2022).
25. Sigelman, B.H.; Barroso, L.A.; Burrows, M.; Stephenson, P.; Plakal, M.; Beaver, D.; Jaspan, S.; Shanbhag, C. Dapper, a Large-Scale
Distributed Systems Tracing Infrastructure; Technical Report; Google, Inc.: Mountain View, CA, USA, 2010.
26. Feldt, R.; Magazinius, A. Validity Threats in Empirical Software Engineering Research-An Initial Survey. In Proceedings of the
SEKE, San Francisco, CA, USA, 1–3 July 2010.
27. Wohlin, C.; Runeson, P.; Höst, M.; Ohlsson, M.C.; Regnell, B.; Wesslén, A., Case Studies. In Experimentation in Software Engineering;
Springer: Berlin/Heidelberg, Germany, 2012; pp. 55–72. https://doi.org/10.1007/978-3-642-29044-2_5.
28. Yin, R. Case Study Research and Applications: Design and Methods; Supplementary Textbook; SAGE Publications: New York, NY,
USA, 2017.
29. Kratzke, N. log12-a Single and Self-Contained Structured Logging Library. 2022. Available online: https://github.com/nkratzke/
log12 (accessed on 20 September 2022).
30. Soldani, J.; Brogi, A. Anomaly Detection and Failure Root Cause Analysis in (Micro) Service-Based Cloud Applications: A Survey.
ACM Comput. Surv. 2022, 55, 1–39. https://doi.org/10.1145/3501297.
31. Zhu, J.; He, S.; Liu, J.; He, P.; Xie, Q.; Zheng, Z.; Lyu, M.R. Tools and benchmarks for automated log parsing. In Proceedings
of the 2019 IEEE/ACM 41st International Conference on Software Engineering: Software Engineering in Practice (ICSE-SEIP),
Montreal, QC, Canada, 25–31 May 2019; pp. 121–130.
32. Oliner, A.; Ganapathi, A.; Xu, W. Advances and challenges in log analysis. Commun. ACM 2012, 55, 55–61.
33. Fu, Q.; Zhu, J.; Hu, W.; Lou, J.G.; Ding, R.; Lin, Q.; Zhang, D.; Xie, T. Where do developers log? an empirical study on logging
practices in industry. In Proceedings of the Companion Proceedings of the 36th International Conference on Software Engineering,
Hyderabad, India, 31 May–7 June 2014; pp. 24–33.
34. Zhu, J.; He, P.; Fu, Q.; Zhang, H.; Lyu, M.R.; Zhang, D. Learning to log: Helping developers make informed logging decisions. In
Proceedings of the 2015 IEEE/ACM 37th IEEE International Conference on Software Engineering, Florence, Italy, 16–24 May 2015;
Volume 1, pp. 415–425.
35. Guan, Q.; Fu, S. Adaptive anomaly identification by exploring metric subspace in cloud computing infrastructures. In Proceedings
of the 2013 IEEE 32nd International Symposium on Reliable Distributed Systems, Braga, Portugal, 1–3 October 2013; pp. 205–214.
36. Pannu, H.S.; Liu, J.; Fu, S. Aad: Adaptive anomaly detection system for cloud computing infrastructures. In Proceedings of the
2012 IEEE 31st Symposium on Reliable Distributed Systems, Irvine, CA, USA, 8–11 October 2012; pp. 396–397.
37. He, S.; Zhu, J.; He, P.; Lyu, M.R. Experience report: System log analysis for anomaly detection. In Proceedings of the 2016 IEEE 27th international symposium on software reliability engineering (ISSRE), Ottawa, ON, Canada , 23–27 October 2016; pp. 207–218.
38. Niedermaier, S.; Koetter, F.; Freymann, A.; Wagner, S. On observability and monitoring of distributed systems–an industry
interview study. In Proceedings of the International Conference on Service-Oriented Computing, Dubai, United Arab Emirates,
14–17 December 2019; Springer: Berlin/Heidelberg, Germany, 2019; pp. 36–52.
39. Marie-Magdelaine, N.; Ahmed, T.; Astruc-Amato, G. Demonstration of an observability framework for cloud native microservices.
In Proceedings of the 2019 IFIP/IEEE Symposium on Integrated Network and Service Management (IM), Bordeaux, France, 18–19
May 2021; pp. 722–724.
40. Picoreti, R.; do Carmo, A.P.; de Queiroz, F.M.; Garcia, A.S.; Vassallo, R.F.; Simeonidou, D. Multilevel observability in cloud
orchestration. InProceedingsofthe2018IEEE16thIntlConfonDependable,AutonomicandSecureComputing,16thIntl Conf on Pervasive Intelligence and Computing, 4th Intl Conf on Big Data Intelligence and Computing and Cyber Science and Technology Congress (DASC/PiCom/DataCom/CyberSciTech), Athens, Greece 12–15 August 2018; pp. 776–784.
41. Karumuri, S.; Solleza, F.; Zdonik, S.; Tatbul, N. Towards observability data management at scale. ACM SIGMOD Rec. 2021, 49, 18–23.
42. Usman, M.; Ferlin, S.; Brunstrom, A.; Taheri, J. A Survey on Observability of Distributed Edge & Container-based Microservices. IEEE Access 2022, 10, 86904–86919. https://doi.org/10.1109/ACCESS.2022.3193102.
43. Chow, M.; Meisner, D.; Flinn, J.; Peek, D.; Wenisch, T.F. The Mystery Machine: End-to-end Performance Analysis of Large-scale Internet Services. In Proceedings of the 11th USENIX Symposium on Operating Systems Design and Implementation (OSDI 14), Carlsbad, CA, USA, 11–13 July 2022; USENIX Association: Broomfield, CO, USA, 2014; pp. 217–231.
44. Janes, A.; Li, X.; Lenarduzzi, V. Open Tracing Tools: Overview and Critical Comparison. arXiv 2022, arXiv:2207.06875.
45. Falcone, Y.; Nazarpour, H.; Jaber, M.; Bozga, M.; Bensalem, S. Tracing distributed component-based systems, a brief overview. In Proceedings of the International Conference on Runtime Verification, Limassol, Cyprus, 10–13 November 2018; Springer: Berlin/Heidelberg, Germany, 2018; pp. 417–425.
46. Bento, A.; Correia, J.; Filipe, R.; Araujo, F.; Cardoso, J. Automated Analysis of Distributed Tracing: Challenges and Research Directions. J. Grid Comput. 2021, 19, 9. https://doi.org/10.1007/s10723-021-09551-5
47. ElasticSearch Authors. ElasticSearch Database. 2022. Available online: https://www.elastic.co/elasticsearch/ (accessed on 20 September 2022).
48. InfluxDB Authors. InfluxDB Time Series Data Platform. 2022. Available online: https://www.influxdata.com/ (accessed on 20 September 2022).
49. Jaeger Authors. Jaeger. 2022. Available online: https://jaegertracing.io (accessed on 20 September 2022).
50. OpenSearch Authors. OpenSearch. 2022. Available online: https://opensearch.org (accessed on 20 September 2022).
51. Prometheus Authors. Prometheus. 2022. Available online: https://prometheus.io (accessed on 20 September 2022).
52. Zipkin Authors. Zipkin. 2022. Available online: https://zipkin.io (accessed on 20 September 2022).
53. Fluentd Authors. Fluentd. 2022. Available online: https://fluentd.org (accessed on 20 September 2022).
54. Flume Authors. Flume. 2022. Available online: https://flume.apache.org (accessed on 20 September 2022).
55. LogStash Authors. LogStash. 2022. Available online: https://www.elastic.co/logstash (accessed on 20 September 2022).
56. FileBeat Authors. FileBeat. 2022. Available online: https://www.elastic.co/filebeat (accessed on 20 September 2022).
57. Rsyslog Authors. RSYSLOG-The Rocket-Fast Syslog Server. 2020. Available online: https://www.rsyslog.com (accessed on 20
September 2022).
58. Syslog-Ng Authors. Syslog-Ng. 2022. Available online: https://www.syslog-ng.com (accessed on 20 September 2022).
59. Go Standard Library Authors. Log. 2022. Available online: https://pkg.go.dev/log (accessed on 20 September 2022).
60. Log4j Authors. Log4j. 2022. Available online: https://logging.apache.org/log4j/2.x (accessed on 20 September 2022).
61. Python Standard Library Authors. Logging. 2022. Available online: https://docs.python.org/3/howto/logging.html (ac-
cessed on 20 September 2022).
62. Micrometer Authors. Micrometer Application Monitor. 2022. Available online: https://micrometer.io/ (accessed on 20
September 2022).
63. Splunk APM Authors. Splunk Application Performance Monitoring. 2022. Available online: https://www.splunk.com/en_us/
products/apm-application-performance-monitoring.html (accessed on 20 September 2022).
64. Schlawack, H. Structlog. 2022. Available online: https://pypi.org/project/structlog (accessed on 20 September 2022).
65. Winston Authors. Winston. 2022. Available online: https://github.com/winstonjs/winston (accessed on 20 September 2022).
66. Linux Foundation. Cloud-Native Computing Foundation, 2015. Available online: https://cncf.io (accessed on 20 September
2022).
67. The OpenTelemetry Authors. The OpenTelemetry Specification-Logs Data Model. 2021. Available online: https://opentelemetry.
io/docs/reference/specification/logs/data-model/ (accessed on 20 September 2022).
68. The OpenTelemetry Authors. The OpenTelemetry Specification-Metrics SDK. 2021. Available online: https://opentelemetry.io/
docs/reference/specification/metrics/sdk/ (accessed on 20 September 2022).
69. The OpenTelemetry Authors. The OpenTelemetry Specification-Tracing SDK. 2021. Available online: https://opentelemetry.io/
docs/reference/specification/trace/sdk/ (accessed on 20 September 2022).
70. The OpenTelemetry Authors. The OpenTelemetry Operator. 2021. Available online: https://github.com/open-telemetry/
opentelemetry-operator (accessed on 20 September 2022).
71. Li, W.; Lemieux, Y.; Gao, J.; Zhao, Z.; Han, Y. Service mesh: Challenges, state of the art, and future research opportunities. In
Proceedings of the 2019 IEEE International Conference on Service-Oriented System Engineering (SOSE), San Francisco, CA, USA,
4–9 April 2019; pp. 122–1225.
72. Malki, A.E.; Zdun, U. Guiding architectural decision making on service mesh based microservice architectures. In Proceedings of
the European Conference on Software Architecture, Paris, France, 9–13 September 2019; Springer: Berlin/Heidelberg, Germany,
2019, pp. 3–19.
73. Service Mesh Interface Authors. SMI: A Standard Interface for Service Meshes on Kubernetes. 2022. Available online:
https://smi-spec.io (accessed on 20 September 2022).
74. Al-Debagy, O.; Martinek, P. A comparative review of microservices and monolithic architectures. In Proceedings of the 2018 IEEE
18th International Symposium on Computational Intelligence and Informatics (CINTI), Budapest, Hungary, 21–22 November
2018; pp. 000149–000154.
75. Balalaie, A.; Heydarnoori, A.; Jamshidi, P.; Tamburri, D.A.; Lynn, T. Microservices migration patterns. Softw. Pract. Exp. 2018,
48, 2019–2042.
76. Sheikh, O.; Dikaleh, S.; Mistry, D.; Pape, D.; Felix, C. Modernize digital applications with microservices management using the
istio service mesh. In Proceedings of the 28th Annual International Conference on Computer Science and Software Engineering,
Toronto, ON, Canada, 29–31 October 2018; pp. 359–360.
77. Istio Authors. The Istio Service Mesh. 2017. Available online: https://istio.io/ (accessed on 20 September 2022).
