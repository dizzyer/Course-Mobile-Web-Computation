# Towards a Personal Mobile Cloud via Generic Device Interconnections

# Abstract

移动计算设备的近期多样化使得移动用户可以拥有多种类型的设备，以适应不同的应用场景，但同时也导致了这些设备在性能和可用性方面的各种限制。解决这些限制的一个可行方案是将移动设备整合并互联，形成一个个人移动云，通过协作资源共享使这些设备相互补充，但由于移动设备在硬件和软件方面的异质性，这一方案颇具挑战。本文提出了一种新颖的资源共享框架设计，以应对这些挑战，并实现异质移动设备的通用互联。我们的基本思路是利用现有的移动操作系统服务作为资源共享的接口，来掩盖移动系统中的硬件和软件异质性，并将资源共享框架进一步发展为移动操作系统中的中间件。我们在具有不同特性和资源限制的各种移动平台上实现了我们的设计，并证明我们的设计能够高效地支持异质移动设备之间的通用资源共享，而不会带来显著的系统开销或需要对单个系统进行修改。

# 1 Introduction

如今，移动用户通常配备多种类型的移动计算设备，从传统的智能手机和平板电脑到新兴的可穿戴设备，每种设备都针对特定的应用场景设计。这种多样化的设计满足了不同应用场景的独特需求，但也限制了这些移动设备在其他方面的性能或可用性。例如，可穿戴设备以小巧的外形实现了身体感知，但在计算、通信和电池寿命方面能力有限。消除这种限制的一个可行解决方案是构建一个个人移动云[15]，通过无线链路[10]、[37]将用户拥有的所有移动设备整合并互联。这些设备随后能够灵活地相互共享系统资源，增强了许多计算密集型应用（如移动虚拟现实[30]、[31]）中为用户提供的移动计算能力。例如，在图1中，当互联时，可穿戴设备和智能手机可以通过利用附近更强大设备（如智能汽车）的计算能力[12]、[27]、[29]或GPS读数[15]、[55]，极大地延长其本地电池寿命。另一方面，智能汽车也可以利用可穿戴设备上各种身体传感器的数据来推断用户的实时行为模式，并相应地做出反应[26]、[45]、[56]。

实现个人移动云的主要挑战在于移动计算设备的异构性，这种异构性体现在硬件和软件两个方面，并阻碍了这些设备以通用方式互联。首先，当今移动设备上搭载的硬件组件种类日益增多，导致这些硬件所使用的驱动程序、I/O 堆栈和数据访问接口存在根本性差异。即使是同类型的硬件，如果硬件驱动由不同制造商提供且彼此不兼容，远程系统访问硬件数据也可能失败。这种不兼容通常是由于不同硬件制造商采用了定制化的 SoC 设计。例如，高通骁龙芯片组的加速度计驱动程序与三星 Exynos 芯片组的驱动程序肯定不兼容。其次，当今移动应用程序的复杂性大幅增加，导致它们对移动系统资源的需求类型及其访问这些资源的具体方式存在异构性。遗憾的是，现有解决方案仅限于针对单个移动应用程序 [38]、[59] 或特定类型的共享硬件 [9]、[47] 来互联移动设备。因此，它们需要大量重编程工作来互联异构移动设备，每次都需要为这些设备的每个硬件或软件组件重新设计。这种重编程工作不仅降低了移动计算系统在多样化环境中的可用性，还增加了移动操作系统的额外开销，从而降低了移动系统性能。

实现异构移动设备之间的通用互连的关键在于开发一种高效的资源共享框架，该框架能够适当地掩盖移动系统中硬件和软件的异构性。然而，由于移动硬件和软件之间的紧密交互，开发这样的资源共享框架具有挑战性。在移动操作系统层次结构的较低层，框架统一了移动应用程序的异构资源请求，但必须处理以本质上不同方式操作的各个硬件驱动程序，因此需要大量的重新工程工作。另一方面，在应用程序层共享系统资源能够通过通用操作系统接口访问移动硬件，但必须与特定的数据传输协议相关联，因此通用性有限。

本文提出了一种移动系统框架，以解决上述挑战，并实现异构移动设备向个人移动云的通用互连。我们的基本思路是将资源共享框架作为移动操作系统中的中间件进行开发，该中间件利用现有的移动操作系统服务在移动设备之间共享资源。这些服务隐藏了设备驱动程序操作的底层细节，同时为用户应用程序提供统一的数据访问API。移动设备之间的互连可以通过远程访问和调用这些操作系统服务来实现。由于这些服务作为操作系统内核的独立系统进程执行，并与应用程序进程分离，因此可以通过移动系统之间的进程间通信（IPC）实现远程服务调用，而无需涉及复杂的内存引用和同步问题。因此，任何新设备都可以通过将其操作系统插入我们的框架中来加入移动云，而无需修改操作系统内核、我们的框架本身或任何移动应用程序的源代码。

我们期望这个框架能够显著改变移动计算系统的设计和运行方式，从而为个人移动云的实际实现做出重大贡献。特别是，我们的框架保留了访问远程系统资源的过程，使其对用户应用程序完全透明，这些应用程序通过本地操作系统提供的相应服务句柄访问这些资源。因此，大量现有的移动系统可以通过增强和采用来构建个人移动云，而无需任何修改或额外的软件重新开发工作。开发者无需处理个别硬件实现和驱动操作的细节，而是能够通过本地设备的通用配置高效地互联多个远程系统组件。因此，个人移动云的开发的可扩展性和可扩展性通过以与本地操作相同的方式操作远程系统资源来得到保证。

我们已经在包括智能手机、平板电脑和智能手表在内的各种移动平台上，使用不到5000行代码（LoC）在Android操作系统上实现了我们的设计，并展示了在远程移动设备之间共享各种类型硬件（GPS、加速度计、音频扬声器、摄像头）的效率。评估结果表明，我们的设计能够高效地支持远程系统之间的普遍资源访问，任意移动应用程序可以访问这些资源，而不会产生任何显著的系统开销。我们提出的框架还与现有的应用程序级远程消息传递协议（例如，MQTT和XMPP）完全兼容，因此也可以高效地用于移动应用程序开发。

本文的其余部分组织如下。在第2节中，我们提供了关于我们的动机和设计的概述。第3节和第4节详细介绍了我们的资源共享框架和应用程序接口。第5节描述了我们如何支持移动设备之间共享多媒体资源。第6节介绍了我们在各种移动平台上的实现，第7节基于这些实现展示了我们的评估结果。第8节讨论了相关工作。最后，第9节讨论了本文的内容，第10节总结了本文。

# 2 Overview

在本节中，我们从简要描述移动操作系统的分层架构开始，这一架构激发了我们提出的设计思路。基于这一动机，我们进一步提供了所提出框架的高层次概述。

## 2.1 Layered Architecture of Mobile OS

由于移动设备的资源有限，移动操作系统通常采用分层架构，以便将用户应用程序与底层系统实现隔离开来。这种隔离使得系统资源管理更加高效，并保护移动系统免受因设计不当的应用程序或移动恶意软件的恶意攻击而导致的资源耗尽。在这种分层操作系统架构中，移动应用程序不直接访问系统资源。相反，资源访问由系统服务通过一组通用且预定义的API提供，这些API分别通过Android中的Binder机制和iOS中的消息传递机制由用户应用程序调用。例如，在Android和iOS中，应用程序不是直接访问GPS或WiFi网络接口，而是通过操作系统提供的位置服务获取设备的位置信息。然后，这些系统服务通过硬件抽象层（HAL）与硬件设备驱动程序进行交互，HAL的接口由操作系统预定义，并由制造商实现为库。

## 2.2 Limitation of the Existing Schemes

现有的移动设备间远程资源共享方案直接在移动应用程序二进制文件或操作系统底层驱动程序上实现，因此无法在移动系统中提供通用且自适应的资源共享。更具体地说，尽管不同应用程序在互联移动系统和在这些系统之间共享资源的方式上具有相似性，但开发这些多样化应用程序的过程仍然非常不同且彼此完全分离。因此，为了远程访问各种移动系统资源，开发者必须在每个应用程序中重新发明轮子，以实现高效互联，并重复实现应用程序包装器来处理异构移动设备之间的通信。例如，一个典型的Android应用程序共享传感器数据和远程播放音频轨道分别需要大约3001和1,1002行代码。这种缺乏通用性的问题已成为阻碍资源共享在实际移动场景中进一步广泛应用的主要障碍。

另一方面，在操作系统驱动层级进行资源共享需要大量的重新编程工作，这是由于移动设备上配备了多种多样的硬件组件。例如，如今移动设备上提供了越来越多的传感器（如加速度计、陀螺仪、指南针等）。为了克服操作单个传感器驱动的复杂性，移动操作系统提供了一个统一的系统服务来访问所有板载传感器。然而，现有的操作系统级资源共享方案未能利用这种统一接口，必须为每个驱动程序编写共享功能。这样做会增加数千行代码来在远程移动系统之间共享所有传感器。

此外，这种重复的编程工作也可能是由于定制的SoC设计以及制造商之间硬件驱动程序的不兼容性所导致的。在这种情况下，资源共享需要重新编程驱动程序实现，以使它们在设备之间保持一致，这通常需要大约350和1,000行代码分别用于典型的加速度计和扬声器驱动程序。需要注意的是，这种重新编程必须针对每对具有不同驱动程序实现的移动设备进行。

支持远程资源共享的另一个关键缺点是在操作系统驱动底层忽略了移动应用程序的运行时执行模式，导致大量数据传输的浪费。更具体地说，底层驱动操作在硬件状态发生变化时同步移动系统之间的资源数据。然而，这种变化频率通常远高于应用程序实际资源请求的频率。例如，每个用户应用程序通常会指定其请求位置数据的间隔，尽管Android中的GPS设备每秒都会报告位置信息。因此，为远程的Google GMS和Facebook应用程序共享GPS设备将分别浪费80%和98%的数据传输，因为它们分别每5秒和60秒请求一次GPS数据。

## 2.3 Our Motivation

我们支持基于上述移动操作系统分层架构的远程移动系统之间的通用资源共享。首先，由于系统服务是用户应用程序访问系统资源的唯一接口，因此只要该框架能够拦截用户应用程序的资源访问请求并将其重定向到远程系统，就可以通过相同的通用框架提供对远程系统上任何类型资源的访问。此外，不同类型的系统服务遵循相同的调用机制（例如，Android 中的 binder 和 iOS 中的消息传递），因此我们不会遇到服务操作的异构性。其次，这些系统服务对用户应用程序隐藏了硬件操作的细节。因此，基于这些服务的资源共享解决了硬件驱动程序实现的异构性，并允许具有不同硬件型号和驱动程序的设备相互访问。更重要的是，由于系统服务是通过预定义的 API 集调用的，因此这些调用的拦截和重定向对用户应用程序是透明的，用户应用程序将以与访问本地资源相同的方式访问远程系统资源。这种透明性因此使得大量现有的移动应用程序能够访问远程资源而无需任何重新编程工作。例如，游戏开发者可以通过以与操作本地设备的 LCD 屏幕相同的方式访问这些外部显示器，轻松地将游戏屏幕投射到外部大屏幕上。同样，活动识别系统的开发者通过假设附近人类用户将有机会提供的各种类型的身体传感器数据，将在其设计中获得更大的灵活性。

## 2.4 The Big Picture

如图2所示，我们提出的中间件位于用户应用程序和操作系统服务之间，由两个主要组件组成：a) 应用程序接口和b) 资源共享框架。

[Fig.2. The big picture of interconnecting heterogeneous mobile devices](./pics/fig2.jpg)

应用程序接口(Application Interface)规定了用户应用程序如何通过一个特定于应用程序的元数据文件访问远程系统的共享资源，该文件配置了远程资源的使用方式。当用户应用程序请求访问系统资源时，应用程序接口解析配置文件以返回相应的系统服务的适当句柄。因此，无论调用的是哪种系统服务，以及该服务是在本地系统还是远程系统上调用，服务都通过相同的方式进行操作。

资源共享框架(Resource Sharing Framework)与本地操作系统交互，并与远程系统服务通信，以向用户应用程序提供远程资源访问。如图2所示，操作系统内核中的底层设备驱动程序实现和硬件操作的细节完全与资源共享框架分离，不同的系统服务通过同一组预定义的API以统一的方式调用。我们的设计策略是通用的，适用于各种主流移动操作系统（如Android、iOS）中的系统服务，这些系统服务采用不同的编程语言和系统架构实现。

# 3 Resource Sharing Framework

我们的资源共享框架设计如图3所示。

[Fig.3. Design of resource sharing framework](./pics/fig3.jpg)

总体而言，我们的框架拦截本地移动应用程序生成的资源访问请求，并将这些请求转发到作为服务器并提供共享资源的远程移动系统。每当接收到资源访问请求时，服务器将调用其本地系统服务以响应请求的资源，并回复资源数据。该框架由两个主要组件组成：代理对象模块和序列化模块。代理对象模块的职责是作为远程服务调用的门户，并管理资源共享系统两端的代理对象。更具体地说，该模块管理两种类型的对象：提供者和请求者。提供者是实现编程接口的对象，也是远程方法调用的目标。该对象注册到代理对象模块以启用自身的远程方法调用。另一方面，请求者是远程提供者的本地代理，并将方法调用请求转发到远程系统。每当移动应用程序在代理对象请求者上发起服务方法调用时，它将被重定向到远程提供者。序列化模块负责数据序列化，即将内存对象转换为可通过网络链接传输的二进制格式。这些二进制文件可以在另一端通过反序列化重建为内存对象。

我们的框架支持移动系统之间的无处不在的资源访问，主要通过两种方式：主动调用和被动回调。首先，当远程系统服务可用时，我们的框架会创建一个服务代理对象，以启动客户端和服务器之间的进程间通信（IPC）。在主动调用中，每当应用程序请求访问远程服务时，客户端的代理对象会触发一个远程调用事件，该事件在服务器端被捕获以调用相应的服务方法。其次，应用程序还可以通过接收系统事件中的数据来被动访问系统资源，例如位置更新。在这种情况下，资源访问由被动回调处理，允许应用程序注册并监听系统事件，并使用回调句柄。当系统事件发生时，服务器端的服务通过回调代理调用该句柄，然后用于将资源数据传输回客户端。这种调用过程类似于主动调用，但方向相反。

## 3.1 Invocation of Remote System Services

在本节中，我们将详细描述我们资源共享框架的设计细节，该框架支持对基于Java和本地系统服务的远程调用。如前所述，这两种类型的系统服务都可以通过主动调用和被动回调进行远程调用。

### 3.1.1 Java-Based System Services

在 Android 中，部分系统服务是用 Java 实现的，并且运行在一个独立的系统进程中。一方面，为了设计一个通用的序列化模块来共享这些服务，我们利用 Java 反射机制来最小化编程工作量，这使得开发者能够在运行时检查类、接口、字段和方法，而无需提前知道类和方法的名称。具体来说，我们通过反射访问任何内存对象的所有字段值，并将它们转换为二进制数据。类似地，这个特性也可以用于反序列化，通过在运行时创建对象实例并设置所有字段值，从而从接收到的二进制数据中重建内存对象。

特别是，序列化和反序列化需要递归遍历内存对象，因为其字段也可能是一个对象。我们将内存对象组织为一个基于树的结构，其中对象本身作为根节点，其字段作为子节点。因此，我们能够使用广度优先搜索遍历对象树来遍历和重建内存对象的内容。此外，当我们遍历内存对象的字段时，如果字段是基本类型，我们就获取或设置其值。否则，字段是对象引用类型，我们在处理完其所有兄弟节点后递归遍历该字段的内容。

另一方面，代理对象模块最直接的解决方案是直接使用 Java 的内置动态代理来实现一个接口，从而利用现有的 Java 反射机制来实例化代理对象。然而，这种方法有根本性的局限性，不能在 Android 中使用，因为 Android 中的用户应用程序必须通过 Android IPC binder 机制访问系统服务，其中用户应用程序和系统服务分别作为 binder 代理和 binder 存根，如图 4 所示。这个机制要求任何能够响应 binder 内核驱动程序调用的 binder 存根必须是 Binder 类的子类。因此，代理对象也必须继承自 Binder 类，以便注册到 binder 内核驱动程序，从而正确拦截应用程序对本地服务 binder 存根的调用。

[Fig.4. Android binder mechanism with the example of location service.](./pics/fig4.jpg)

我们的方法是通过从现有系统服务类中开发代理对象，通过动态编织将远程调用操作的字节码附加到服务类中。Java中的动态编织技术允许我们在运行时对现有Java类的字节码进行插桩，生成一个作为原始类类型子类的新类实例。由于Android中的系统服务类本身被指定为Binder类的子类，因此每当系统服务类将被远程访问时，我们都会在运行时动态编织该类，并在服务方法中添加额外的远程调用操作字节码。因此，这个新编织的类将是Android Binder类的子类，并且可以注册到Binder内核驱动程序中，以接收和拦截应用程序对服务方法的调用。

[Fig.5. Resource sharing through callbacks.](./pics/fig5.jpg)

除了应用程序主动调用远程系统服务的主动调用外，我们的框架还支持回调到注册了系统事件的用户应用程序，这些应用程序使用任何实现了事件监听器接口的类对象。然后，如图5所示，用户应用程序利用Android系统库将此监听器包装成一个Binder桩，该桩与系统服务中的Binder代理进行通信以监听事件。为了实现两个移动设备之间的这种回调，如图3所示，我们允许应用程序通过回调代理在远程系统中注册并监听事件，并利用客户端的事件监听器Binder代理作为回调句柄。之后，当系统事件在服务器端发生时，回调代理将发起对客户端回调句柄的远程调用。

### 3.1.2 Native System Services

现有移动操作系统中另一大类系统服务是用原生C/C++语言实现的，例如Android中的传感器和图形服务，以及iOS中的所有系统服务。由于这些服务在运行时作为编译后的二进制文件执行，因此无法通过运行时操作来开发这些服务的代理对象，原因如下。首先，很难在运行时定位原生方法的入口和出口点，因此难以动态地将编织指令附加到服务类上。其次，这些原生服务类中的机器指令依赖于硬件架构，因此只能静态地操作和编译。

[Fig.6. Modification of native service codes with the sensor service as an example.](./pics/fig6.jpg)

为了应对这一挑战，在我们当前的设计中，我们手动修改每个系统服务类的源代码，以实现序列化和反序列化功能，以及服务方法的远程调用。以图6中的Android传感器服务为例，我们实现了一个请求者和一个提供者，它们分别与用户应用程序和系统服务进行交互。请求者序列化参数，调用远程服务方法，并反序列化以获取调用结果。提供者反序列化以获取参数，调用本地传感器服务以启用传感器，并序列化调用结果。这些源代码随后与其他操作系统可执行文件一起编译。

| Service | Sensor Service | Audio Service | Camera Service |
| Serialization | 63 | 114 | 137|
| Invocation | 51 | 187 | 109 |

我们已经定量测量了这种手动修改的数量，以实现远程调用的系统服务子类的代码行数（LoC）为单位。如表1所示，我们的框架在每个系统服务上产生的工程开销很小，不到300行代码。需要注意的是，这种手动修改的方法是通用的，适用于所有其他操作系统服务。与现有方案不同，现有方案通过设备驱动程序共享系统资源，并且必须为每个移动系统重新编程驱动程序实现，而我们的修改是一次性工作，适用于所有具有异构硬件组件的移动系统。这一优势使我们的工作能够应用于各种不同的移动平台，我们将在第6节中描述这些实现细节。

### 3.1.3 Supporting Concurrent Resource Sharing

**共享多个资源并发**

不同类型的资源可能在两个移动设备之间同时共享，导致多个服务方法的并发调用。为了支持这种并发资源共享，每个IPC提供者必须监听正确的调用事件。否则，调用错误的提供者对象将导致严重的运行时错误。

[Fig.7. Identifier mapping in the proxy object module.](./pics/fig7.jpg)

我们确保提供者和请求者之间调用一致性的方法是标识符映射方案。在代理对象模块中维护一个索引表，其中包含标识符与提供者之间的映射。当一个提供者向代理对象模块注册时，将在索引表中添加一个映射条目。此外，当请求者向远程提供者发出请求时，请求消息中将包含一个标识符，指定目标提供者。另一端的代理对象模块将根据接收到的标识符查找提供者，并对该提供者进行方法调用。例如，在图7中，位置服务和通知服务分别使用标识符1和2向代理对象模块注册。当共享客户端的请求者发出带有标识符1的方法调用时，共享服务器端的代理对象模块将索引映射表并将方法调用转发给位置服务。在主动调用中，提供者是系统服务，我们预先为每个系统服务分配一个标识符，并在共享服务器和客户端之间保持一致，因为系统服务的列表是预先已知的。另一方面，反应式回调需要动态标识符分配方案，因为事件监听对象的出现是不可预测的，并且依赖于移动应用程序的运行时执行模式。

**并发访问同一资源**

本地移动应用程序和多个共享客户端可能同时访问特定资源。对于固有支持多个本地应用程序的系统服务（如位置服务和传感器服务），可以通过上述标识符映射方案轻松实现并发访问，其中每个共享客户端以与本地应用程序相同的方式进行管理。相比之下，多媒体系统服务设计为仅由一个移动应用程序独占访问，并且禁止对这些资源的并发访问。当新的共享客户端需要访问已被占用的多媒体资源时，我们的框架允许移动用户通过应用程序接口组件中的配置（参见第4节）决定是等待还是抢占资源访问。


**移动设备同时作为共享服务器和客户端**

尽管我们的资源共享基于客户端-服务器结构，但我们的框架允许移动设备自由地同时充当共享服务器和共享客户端。更具体地说，我们的框架设计了单独的守护线程来表示共享服务器和客户端，以进行远程交互。作为共享服务器的守护线程处理远程资源访问请求，并与相应的系统服务交互以提供资源数据。另一方面，作为共享客户端的守护线程连接到共享服务器，并为远程调用系统服务创建相应的服务代理。在移动系统启动期间，我们的框架读取用户配置并启动相应的守护线程。

## 3.2 Unix Domain Socket

我们框架中支持的另一种IPC方案是Unix域套接字。例如在Android中，传感器数据并不是通过Binder方法调用的参数从传感器服务传递给应用程序的。相反，它是通过传感器服务和应用程序之间的Unix域套接字传递的，以减少频繁数据传输的系统开销。在我们的设计中，我们使用网络套接字而不是现有IPC方案中使用的Unix域套接字来构建移动设备之间的可靠数据交换通道。更具体地说，服务器中的系统服务进程将监听建立分布式数据通道的连接请求。当应用程序请求远程系统服务时，客户端中的代理对象将建立与服务器的连接，并将该连接的文件描述符传递回应用程序。此后，服务器中的系统服务可以通过TCP与客户端应用程序可靠地交换数据。由于移动操作系统使用相同的API来操作这两种套接字，因此数据通道套接字的实际类型可以对使用数据通道的系统隐藏，并且用于数据交换的现有IPC代码可以保持不变。

## 3.3 Error Handling

我们提出的资源共享框架需要确保即使在发生意外系统错误并影响远程资源访问的情况下，应用程序仍能正确执行。如图8所示，我们将可能的系统错误分为两类，并采取不同的策略应对。首先，对于可恢复的系统错误（例如，不稳定的网络连接），我们的框架会在错误恢复后重新连接到远程共享服务器，并恢复使用远程资源。更具体地说，客户端的框架会定期向服务器发送信标。如果信标超时，客户端的框架会发起与服务器的重新连接。

[Fig.8. Error Handling in the Sharing Framework](./pics/fig8.jpg)

另一方面，对于不可恢复的系统错误，如资源运行时故障和不可用性，我们的框架选择回退到本地资源访问。更具体地说，共享服务器在资源访问期间监听运行时错误，并向操作系统内核注册信号处理程序，该程序会在终止资源共享之前通知共享客户端。同时，在调用远程系统服务期间，我们的框架会记录移动应用程序访问远程资源时使用的参数。当客户端收到不可恢复的系统错误通知时，框架会使用记录的参数调用本地系统服务，并将本地资源数据返回给移动应用程序。

# 4 Application Interface

我们的应用程序接口设计如图9所示。基本上，每当用户应用程序请求访问系统服务时，应用程序接口会拦截该请求并返回适当的系统服务句柄，该句柄可能位于本地系统或远程系统中。由于这两个句柄都向应用程序提供了相同的编程接口，因此移动系统之间资源共享的过程对用户应用程序是完全透明的。

[Fig.9. Design of Application Interface](./pics/fig9.jpg)

资源访问的迁移决策以及随后返回的服务句柄是基于存储在应用程序目录中的应用程序特定元数据文件中的配置来确定的。更具体地说，当用户应用程序请求访问系统服务时，框架会从元数据文件中加载配置。如果配置指示将访问本地系统资源，则将本地服务句柄返回给用户。否则，我们的框架将创建一个远程服务代理，并建立与远程系统的TCP连接以迁移资源访问。在我们的设计中，这个元数据文件由一个特殊的代理应用程序操作，该应用程序作为应用程序接口的一部分嵌入。该代理应用程序管理和覆盖所有用户应用程序的资源访问配置，并接收来自移动操作系统设置的可用系统资源列表的输入。所有这些信息都将由代理应用程序写入元数据文件，然后在运行时由我们的框架检查，以确保正确的服务调用。

通过开发这个代理应用程序，我们的设计允许用户应用程序以三种方式配置其系统资源访问的迁移决策。首先，我们允许开发者在安装期间将其配置与应用程序二进制文件一起分发，并明确指定应用程序将如何访问系统资源。例如，开发者可以决定远程资源的目标位置以及他们希望访问远程资源的数据速率。其次，如果资源共享的目标未知，开发者可以选择采用现有的服务发现协议（例如，[49]，[60]），并通过在配置中指定所使用的服务发现协议来探索附近的可用共享资源。第三，我们还允许移动用户通过代理应用程序手动修改资源共享的配置。例如，移动用户可以明确配置应用程序将屏幕内容投影到附近的大尺寸液晶显示器上。

# 5 Supporting Multimedia Operations

多媒体资源是移动设备中的关键组成部分。例如，我们可能将音频从移动设备播放到附近音质更好的扬声器，或者通过移动设备访问周围的摄像头监控系统，以便更好地了解周围环境。然而，多媒体资源的操作通常涉及大量数据，并且需要利用共享内存来实现应用程序之间的数据交换，这给远程系统之间共享这些资源带来了新的挑战。在本节中，我们将介绍如何在远程移动系统之间支持访问多媒体服务的共享内存的设计。然而，主要的挑战在于如何在不改变原始移动操作系统结构和接口的情况下，确保远程进程间通信（IPC）的可靠性。

根据共享内存的操作方式，我们将共享内存分为两种情况并分别处理：通用缓冲区和图形缓冲区。通用缓冲区被定义为可移植且与供应商无关的共享内存。它由系统服务直接分配和操作。而图形缓冲区则存储图像数据，如摄像头预览和视频帧，通常由供应商特定的硬件抽象层（HAL）操作。

## 5.1 The General Buffer

通用缓冲区可以分为两部分。首先，内容部分(Content Part)存储资源数据，并遵循生产者-消费者模式进行操作，即一个操作符总是将数据写入缓冲区，而另一个操作符总是读取数据。通过这种方式，两个操作方始终保持同步，避免了写入时的争用。其次，控制部分(Control Part)存储操作内容部分所需的控制信息，例如读写指针。因此，控制部分可以由应用程序和系统服务共同写入，这可能会在写入时发生冲突。在本节中，我们开发了不同的技术来操作这两个部分，并采用了写更新和写失效协议[43]来实现共享内存的同步。这些内存同步协议在图10中有详细说明。在写更新协议中，每次本地写入后都会立即发送同步消息，以始终保持本地和远程内存之间的一致性。在写失效协议中，远程内存将保留旧值并带有失效标志，该标志由本地写入前发送的失效消息设置。新值将在本地读取发生之前不会同步。我们采用写更新协议来同步频繁读取的内存，并节省失效消息的开销。我们采用写失效协议来同步不频繁读取的内存，以最大化内存同步的效率。

[Fig.10. Memory synchronization protocols](./pics/fig10.jpg)

### 5.1.1 Content Part

为了高效地操作内容部分，与传统的分布式共享内存（DSM）[25]、[50] 方法不同，这些方法通常以固定大小的单元共享内存，并且可能导致移动系统之间大量冗余的数据同步，我们根据内存访问的实际应用模式，灵活地以任意大小同步共享内存。这种灵活性主要是因为在共享客户端和服务器同步内容部分时，它们之间没有写入争用，确保任何大小的同步内存始终保持一致。因此，我们的基本想法是在客户端和服务器之间建立内存映射，并基于该映射同步内存内容。每当一个端点分配一块共享内存时，另一端点也会相应地分配相同大小的内存块，并在两端点维护的映射表中添加它们地址之间的映射条目。每当一个端点完成其写操作时，我们使用映射条目和写入偏移量来计算目标写入地址并同步被写入的数据。例如，在图11中，当客户端分配12字节的内存时，服务器也会相应地分配相同数量的内存，并且客户端和服务器内存的地址都存储在两端点维护的映射表中。之后，当客户端向缓冲区写入5字节时，资源共享框架根据接收到的目标地址和偏移量，同步并更新适当大小的内存。

[Fig.11. Memory synchronization for content part](./pics/fig11.jpg)

主要地，我们采用写更新协议来同步内容部分。然而，通过维护无效内存字段列表，也可以采用写失效协议。更具体地说，在写操作期间，被写入的共享内存地址集合被标记为无效，并添加到远程系统维护的列表中。当另一端点读取内存时，我们的框架根据维护的列表同步内存。

### 5.1.2 Control Part

我们还为控制部分开发了一种灵活的同步方案，可以同步整个控制部分或其中的各个字段。为了确保在写入竞争情况下的数据一致性，我们通过所有权标志在两个移动设备之间建立了一个“先发生”关系，并且只允许内存字段的所有者对该字段进行写入。当另一端尝试写入该字段时，所有权会发生转移。通过这种方式，我们确保了客户端和服务器之间的写入操作按顺序进行，从而使得两个端点的内存始终保持一致。

[Fig.12. Memory synchronization for control part](./pics/fig12.jpg)

例如，在图12中，客户端打算写入一个控制字段，但没有该字段的所有权。因此，客户端向服务器发送一条消息，请求获得该字段的所有权。服务器接收到此消息后，将其所有权转移给客户端。特别地，内存同步消息与所有权请求一起发送，以减少往返次数。我们的框架支持控制部分的写入-更新协议和写入-失效协议。应用程序和系统服务都能够通过一个通用API为各个内存字段选择同步协议，该API在共享内存分配期间注册控制部分的同步元数据。这些元数据随后用于指示我们的框架执行定制的内存同步。

### 5.1.3 Optimization

在实践中，我们观察到对内容部分的写入通常紧随其后的是对控制部分的更新。例如，应用程序将数据写入内容缓冲区后，会更新控制部分中的下一个写入位置。因此，我们进一步优化了内存同步方法，允许系统服务同时同步内容部分和控制部分。系统服务可以延迟内容部分的同步，直到相应的控制部分开始同步。然后，内容数据可以与控制部分的同步消息一起发送，从而减少了通过无线链接进行多次往返的额外开销。

## 5.2 Graphic Buffer

移动操作系统中的图形服务，如操作摄像头或LCD屏幕等多媒体设备，通常利用GPU来加速图像处理和渲染的速度。因此，与应用程序或系统服务自行分配和写入的通用缓冲区不同，图形缓冲区由供应商特定的HAL（硬件抽象层）分配和操作。因此，我们无法直接从资源共享框架中拦截缓冲区操作，并进一步为远程系统之间的同步建立内存映射。相反，我们的方法是将与操作系统提供的用于用户应用程序管理和操作图形缓冲区的API集成到我们的资源共享框架中。

在不失一般性的前提下，我们以Android操作系统为例，详细介绍我们的设计。在Android中，图形服务的核心是BufferQueue类，它管理由供应商特定的gralloc模块分配的不同图形缓冲区。图形缓冲区的操作也遵循生产者-消费者模式：生产者（通常是硬件设备驱动程序）从BufferQueue中出队一个空缓冲区句柄，并将填充好的缓冲区重新入队到BufferQueue；消费者（例如，Surface Flinger）从BufferQueue获取填充缓冲区的句柄，并释放已消费的缓冲区。

[Fig.13. Operating the graphic buffer for remote camera access.](./pics/fig13.jpg)

因此，我们的资源共享框架充当BufferQueue的消费者，提取图形缓冲区的内容，然后将这些内容推送到远程设备。例如，在图13中，服务器中的摄像头驱动程序将预览图像缓冲区发布到BufferQueue后，我们的框架作为消费者收集图形缓冲区内容，并将其发送到客户端。然后，客户端的框架检索一个空缓冲区，用接收到的图像内容填充缓冲区，并将缓冲区重新发布到BufferQueue。之后，客户端的Surface Flinger在摄像头应用程序中渲染预览图像。
