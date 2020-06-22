---
title: Considerações de design de aplicativos híbridos no Azure e Azure Stack Hub
description: Saiba mais sobre considerações de design ao construir uma app híbrida para a nuvem inteligente e borda inteligente, incluindo colocação, escalabilidade, disponibilidade e resiliência.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4fd52f76baad8059e130adfc01cdd0152b40a510
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911111"
---
# <a name="hybrid-app-design-considerations"></a>Considerações híbridas de design de aplicativos

O Microsoft Azure é a única nuvem híbrida consistente. Permite-lhe reutilizar os seus investimentos de desenvolvimento e permite aplicações que podem abranger o Azure global, as nuvens soberanas do Azure, e a Azure Stack, que é uma extensão do Azure no seu datacenter. As aplicações que abrangem nuvens também são referidas como *aplicações híbridas.*

O [*Guia de Arquitetura de Aplicações Azure*](https://docs.microsoft.com/azure/architecture/guide) descreve uma abordagem estruturada para a conceção de apps que são escaláveis, resistentes e altamente disponíveis. As considerações descritas no Guia de Arquitetura de [*Aplicações Azure*](https://docs.microsoft.com/azure/architecture/guide) aplicam-se igualmente a aplicações concebidas para uma única nuvem e para apps que se estendem por nuvens.

Este artigo aumenta os [*Pilares da qualidade do software discutidos*](https://docs.microsoft.com/azure/architecture/guide/pillars) no Guia de [ *Arquitetura*](https://docs.microsoft.com/azure/architecture/guide/) de [*Aplicações Azure,*](https://docs.microsoft.com/azure/architecture/guide/) focando-se especificamente na conceção de aplicações híbridas. Além disso, adicionamos um pilar *de colocação,* uma vez que as aplicações híbridas não são exclusivas de uma nuvem ou de um datacenter no local.

Os cenários híbridos variam muito com os recursos disponíveis para o desenvolvimento, e abrangem considerações como geografia, segurança, acesso à Internet e outras considerações. Embora este guia não possa enumerar as suas considerações específicas, pode fornecer algumas diretrizes e boas práticas para seguir. Projetar, configurar, implementar e manter uma arquitetura híbrida de aplicações envolve muitas considerações de design que podem não ser inerentemente conhecidas por si.

Este documento visa agregar as possíveis questões que possam surgir na implementação de aplicações híbridas e fornece considerações (estes pilares) e boas práticas para trabalhar com elas. Ao abordar estas questões durante a fase de design, evitará os problemas que podem causar na produção.

Essencialmente, estas são questões em que precisa de pensar antes de criar uma aplicação híbrida. Para começar, é preciso fazer as seguintes coisas:

- Identificar e avaliar os componentes da aplicação.
- Avalie os componentes da aplicação contra os pilares.

## <a name="evaluate-the-app-components"></a>Avaliar os componentes da aplicação

Cada componente de uma aplicação tem o seu próprio papel específico dentro da app maior e deve ser revisto com todas as considerações de design. Os requisitos e funcionalidades de cada componente devem mapear estas considerações para ajudar a determinar a arquitetura da aplicação.

Decomponha a sua aplicação nos seus componentes, estudando a arquitetura da sua aplicação e determinando o que consiste. Os componentes também podem incluir outras aplicações com as quais a sua aplicação interage. Ao identificar os componentes, avalie as suas operações híbridas pretendidas de acordo com as suas características, fazendo estas perguntas:

- Qual é o propósito do componente?
- Quais são as interdependências entre os componentes?

Por exemplo, uma aplicação pode ter uma extremidade frontal e traseira definida como dois componentes. Num cenário híbrido, a parte frontal está numa nuvem e a parte de trás está na outra. A aplicação fornece canais de comunicação entre a parte frontal e o utilizador, e também entre a extremidade frontal e a extremidade traseira.

Um componente de aplicação é definido por muitas formas e cenários. A tarefa mais importante é identificá-los e a sua nuvem ou localização no local.

Os componentes comuns da aplicação a incluir no seu inventário estão listados na Tabela 1.

### <a name="table-1-common-app-components"></a>Tabela 1. Componentes comuns de aplicativos

| **Componente** | **Orientação híbrida da aplicação** |
| ---- | ---- |
| Ligações de cliente | A sua aplicação (em qualquer dispositivo) pode aceder aos utilizadores de várias formas, a partir de um ponto de entrada única, incluindo as seguintes formas:<br>- Um modelo de servidor de clientes que exige que o utilizador tenha um cliente instalado para trabalhar com a aplicação. Uma aplicação baseada em servidores que é acedida a partir de um navegador.<br>- As ligações do cliente podem incluir notificações quando a ligação é quebrada ou alertas quando as taxas de roaming podem ser aplicadas. |
| Autenticação  | A autenticação pode ser necessária para um utilizador que se conecte à aplicação ou de um componente que se ligue a outro. |
| APIs  | Pode fornecer aos desenvolvedores acesso programático à sua aplicação com conjuntos de API e bibliotecas de classe e fornecer uma interface de conexão baseada nos padrões de internet. Também pode usar APIs para decompor uma aplicação em unidades lógicas de funcionamento independente. |
| Serviços  | Pode utilizar serviços sucintas para fornecer as funcionalidades de uma aplicação. Um serviço pode ser o motor em que a aplicação funciona. |
| Filas | Pode utilizar as filas para organizar o estado dos ciclos de vida e estados dos componentes da sua aplicação. Estas filas podem fornecer mensagens, notificações e capacidades de tampão para as partes que subscrevem. |
| Armazenamento de dados | Uma aplicação pode ser apátrida ou imponente. As aplicações imponentes precisam de armazenamento de dados que podem ser cumpridos por inúmeros formatos e volumes. |
| Colocação de dados em cache  | Um componente de caching de dados no seu design pode estrategicamente abordar problemas de latência e desempenhar um papel no desencadear a explosão de nuvens. |
| Ingestão de dados | Os dados podem ser submetidos a uma aplicação de muitas formas, desde valores submetidos pelo utilizador num formulário web até ao fluxo de dados continuamente elevado. |
| Processamento de dados | As suas tarefas de processamento de dados (tais como relatórios, análises, exportações de lotes e transformação de dados) podem ser processadas na fonte ou descarregadas num componente separado utilizando uma cópia dos dados. |

## <a name="assess-app-components-for-pillars"></a>Avaliar componentes de aplicativos para pilares

Para cada componente, avalie as suas características para cada pilar. À medida que avalia cada componente com todos os pilares, as questões que talvez não tenha considerado podem tornar-se conhecidas que afetam o design da app híbrida. Agir sobre estas considerações poderia acrescentar valor na otimização da sua app. O quadro 2 apresenta uma descrição de cada pilar no que diz respeito a aplicações híbridas.

### <a name="table-2-pillars"></a>Tabela 2. Pilares

| **Pilar** | **Descrição** |
| ----------- | --------------------------------------------------------- |
| Posicionamento  | O posicionamento estratégico dos componentes em aplicações híbridas. |
| Escalabilidade  | A capacidade de um sistema de processar um aumento de carga. |
| Disponibilidade  | A proporção de tempo que uma aplicação híbrida é funcional e funcional. |
| Resiliência | A capacidade de uma aplicação híbrida recuperar. |
| Capacidade de gestão | Os processos de operações que mantêm um sistema em execução na produção. |
| Segurança | Proteger aplicativos híbridos e dados contra ameaças. |

## <a name="placement"></a>Posicionamento

Uma aplicação híbrida tem inerentemente uma consideração de colocação, como para o datacenter.

A colocação é a tarefa importante de posicionar componentes para que possam melhor servir uma aplicação híbrida. Por definição, as aplicações híbridas abrangem locais, como desde as instalações até à nuvem e entre diferentes nuvens. Pode colocar componentes da aplicação em nuvens de duas formas:

- **Aplicativos híbridos verticais**  
    Os componentes da aplicação são distribuídos por locais. Cada componente individual pode ter múltiplas instâncias localizadas apenas num único local.

- **Aplicativos híbridos horizontais**  
    Os componentes da aplicação são distribuídos por locais. Cada componente individual pode ter múltiplas instâncias abrangendo vários locais.

    Alguns componentes podem estar cientes da sua localização enquanto outros não têm qualquer conhecimento da sua localização e colocação. Esta virtuosidade pode ser alcançada com uma camada de abstração. Esta camada, com uma estrutura de aplicações moderna como microserviços, pode definir como a aplicação é servida pela colocação de componentes de aplicações que operam em nós através de nuvens.

### <a name="placement-checklist"></a>Lista de verificação de colocação

**Verifique os locais necessários.** Certifique-se de que a aplicação ou qualquer um dos seus componentes são necessários para operar ou exigir certificação para uma nuvem específica. Isto pode incluir requisitos de soberania da sua empresa ou ditados por lei. Além disso, determine se são necessárias quaisquer operações no local para um determinado local ou local.

**Verificar dependências de conectividade.** Locais necessários e outros fatores podem ditar as dependências de conectividade entre os seus componentes. Ao colocar os componentes, determine a conectividade e segurança ideais para a comunicação entre eles. As escolhas incluem [ *VPN,*](https://docs.microsoft.com/azure/vpn-gateway/) [ *ExpressRoute*](https://docs.microsoft.com/azure/expressroute/) e [ *Conexões Híbridas.*](https://docs.microsoft.com/azure/app-service/app-service-hybrid-connections)

**Avaliar as capacidades da plataforma.** Para cada componente da aplicação, consulte se o fornecedor de recursos necessário para o componente da aplicação está disponível na nuvem e se a largura de banda pode acomodar os requisitos de produção e latência esperados.

**Plano de portabilidade.** Utilize quadros de aplicações modernos, como contentores ou microserviços, para planear operações em movimento e prevenir dependências de serviços.

**Determine os requisitos de soberania de dados.** As aplicações híbridas são direcionadas para acomodar o isolamento de dados, como num datacenter local. Reveja a colocação dos seus recursos para otimizar o sucesso para acomodar este requisito.

**Plano de latência.** As operações inter-nuvem podem introduzir distância física entre os componentes da aplicação. Verificar os requisitos para acomodar qualquer latência.

**Controlar os fluxos de tráfego.** Manuseie o uso do pico e as comunicações adequadas e seguras para dados pessoais de informação identificáveis quando acedidos pela parte frontal numa nuvem pública.

## <a name="scalability"></a>Escalabilidade

A escalabilidade é a capacidade de um sistema lidar com o aumento da carga numa aplicação, que pode variar ao longo do tempo, uma vez que outros fatores e forças afetam o tamanho do público, além do tamanho e âmbito da app.

Para a discussão central deste pilar, [*veja-se a Escalabilidade*](https://docs.microsoft.com/azure/architecture/guide/pillars#scalability) nos cinco pilares da excelência da arquitetura.

Uma abordagem de escala horizontal para apps híbridas permite adicionar mais instâncias para satisfazer a procura e, em seguida, desativá-las durante períodos mais silenciosos.

Em cenários híbridos, a escala de componentes individuais requer consideração adicional quando os componentes são espalhados pelas nuvens. Escalar uma parte da app pode exigir o escalonamento de outra. Por exemplo, se o número de ligações ao cliente aumentar, mas os serviços web da aplicação não forem dimensionado adequadamente, a carga na base de dados pode saturar a aplicação.

Alguns componentes de aplicações podem escalar linearmente, enquanto outros têm dependências de escala e podem estar limitados até que ponto são capazes de escalar. Por exemplo, um túnel VPN que fornece conectividade híbrida para os locais dos componentes da aplicação tem um limite para a largura de banda e latência a que pode ser dimensionado. Como são os componentes da aplicação dimensionado para garantir que estes requisitos são cumpridos?

### <a name="scalability-checklist"></a>Lista de verificação de escalabilidade

**Verificar limiares de escala.** Para lidar com as várias dependências da sua app, determine até que ponto os componentes da aplicação em diferentes nuvens podem escalar independentemente uns dos outros, cumprindo ainda os requisitos para executar a app. As aplicações híbridas muitas vezes precisam de escalar áreas específicas na app para lidar com uma funcionalidade, uma vez que interage e afeta o resto da app. Por exemplo, exceder uma série de instâncias frontais pode exigir a escalonamento da extremidade traseira.

**Defina horários de escala.** A maioria das aplicações tem períodos ocupados, por isso é necessário agregar os seus tempos de pico em horários para coordenar o escalonamento ideal.

**Utilize um sistema de monitorização centralizado.** As capacidades de monitorização da plataforma podem fornecer autoscaling, mas as aplicações híbridas precisam de um sistema de monitorização centralizado que agrega a saúde e a carga do sistema. Um sistema de monitorização centralizado pode iniciar a escala de um recurso num local e a escala, dependendo do recurso em outro local. Além disso, um sistema central de monitorização pode rastrear quais nuvens de recursos de autoescala e quais nuvens não.

**Aproveite as capacidades de autoscalagem (conforme disponível).** Se as capacidades de autoscalagem fazem parte da sua arquitetura, implementa a autoscalagem estabelecendo limiares que definem quando um componente de aplicação precisa de ser dimensionado, para fora, para baixo ou para dentro. Um exemplo de autoscaling é uma ligação ao cliente que é autoescalada numa nuvem para lidar com o aumento da capacidade, mas que faz com que outras dependências da app, espalhadas por diferentes nuvens, também sejam dimensionadas. As capacidades de autoescalagem destes componentes dependentes devem ser apuradas.

Se não estiver disponível a autoscalagem, considere implementar scripts e outros recursos para acomodar a escala manual, desencadeada por limiares no sistema de monitorização centralizado.

**Determine a carga esperada por localização.** As aplicações híbridas que lidam com os pedidos dos clientes podem depender principalmente de uma única localização. Quando a carga de pedidos do cliente excede um limiar, os recursos adicionais podem ser adicionados num local diferente para distribuir a carga de pedidos de entrada. Certifique-se de que as ligações do cliente podem lidar com as cargas aumentadas e também determine quaisquer procedimentos automatizados para que as ligações do cliente manuseiem a carga.

## <a name="availability"></a>Disponibilidade

Disponibilidade é o tempo que um sistema está funcional e funcionando. A disponibilidade é medida em percentagem de tempo de paragem. Erros de aplicações, problemas de infraestrutura e carga do sistema podem reduzir a disponibilidade.

Para a discussão central deste pilar, veja [*Disponibilidade*](/azure/architecture/framework/) nos cinco pilares da excelência da arquitetura.

### <a name="availability-checklist"></a>Lista de verificação de disponibilidade

**Providenciar redundância para conectividade.** As aplicações híbridas requerem conectividade entre as nuvens que a aplicação está espalhada. Você tem uma escolha de tecnologias para conectividade híbrida, por isso, além da sua escolha de tecnologia primária, use outra tecnologia para fornecer redundância com capacidades automatizadas de failover caso a tecnologia primária falhe.

**Classifique os domínios de avaria.** Aplicações tolerantes a falhas requerem vários domínios de falha. Os domínios de avaria ajudam a isolar o ponto de falha, como se um único disco rígido falha nas instalações, se um interruptor topo de cremalheira se desligar ou se o centro de dados completo não estiver disponível. Numa aplicação híbrida, um local pode ser classificado como um domínio de falha. Com mais requisitos de disponibilidade, mais precisa avaliar como um único domínio de falha deve ser classificado.

**Classifique os domínios de upgrade.** Os domínios de upgrade são utilizados para garantir que estão disponíveis casos de componentes de aplicações, enquanto outras instâncias do mesmo componente estão a ser servidas com atualizações ou atualizações de funcionalidades. Tal como acontece com os domínios de avaria, os domínios de atualização podem ser classificados pela sua colocação em locais. Tem de determinar se um componente da aplicação pode acomodar a atualização num local antes de ser atualizado noutro local ou se são necessárias outras configurações de domínio. Uma única localização em si pode ter vários domínios de upgrade.

**Rastrear instâncias e disponibilidade.** Componentes de aplicações altamente disponíveis podem estar disponíveis através do equilíbrio de carga e replicação de dados sincronizados. Tem de determinar quantas ocorrências podem estar offline antes de o serviço ser interrompido.

**Implementar a auto-cura.** No caso de um problema causar uma interrupção na disponibilidade da aplicação, uma deteção por um sistema de monitorização poderia iniciar atividades de autorrecuperação para a app, como a drenagem da instância falhada e a sua redistribuição. Muito provavelmente isto requer uma solução central de monitorização, integrada com um pipeline híbrido de integração contínua e entrega contínua (CI/CD). A aplicação está integrada num sistema de monitorização para identificar problemas que possam necessitar de reafectação de um componente da app. O sistema de monitorização também pode desencadear ci/CD híbrido para redistribuir o componente da aplicação e potencialmente quaisquer outros componentes dependentes nos mesmos locais ou noutros locais.

**Manter acordos de nível de serviço (SLAs).** A disponibilidade é fundamental para quaisquer acordos para manter a conectividade com os serviços e aplicações que tem com os seus clientes. Cada local em que a sua aplicação híbrida se baseia pode ter o seu próprio SLA. Estes diferentes SLAs podem afetar o SLA geral da sua aplicação híbrida.

## <a name="resiliency"></a>Resiliência

Resiliência é a capacidade de uma aplicação e sistema híbridos recuperarem de falhas e continuarem a funcionar. O objetivo da resiliência é devolver a app a um estado em pleno funcionamento depois de ocorrer uma falha. As estratégias de resiliência incluem soluções como backup, replicação e recuperação de desastres.

Para a discussão central deste pilar, [*veja-se a Resiliência*](https://docs.microsoft.com/azure/architecture/guide/pillars#resiliency) nos cinco pilares da excelência da arquitetura.

### <a name="resiliency-checklist"></a>Lista de verificação da resiliência

**Descubra as dependências de recuperação de desastres.** A recuperação de desastres numa nuvem pode exigir alterações aos componentes da aplicação noutra nuvem. Se um ou vários componentes de uma nuvem forem falhados para outro local, dentro da mesma nuvem ou para outra nuvem, os componentes dependentes precisam de ser sensibilizados para estas alterações. Isto também inclui as dependências de conectividade. A resiliência requer um plano de recuperação de aplicações totalmente testado para cada nuvem.

**Estabelecer fluxo de recuperação.** Um design eficaz do fluxo de recuperação avaliou componentes de aplicações pela sua capacidade de acomodar tampões, retréis, repreensão da transferência de dados falhada e, se necessário, voltar a um serviço ou fluxo de trabalho diferente. Deve determinar que mecanismo de back-up usar, qual o seu procedimento de restauro, e quantas vezes é testado. Também deve determinar a frequência para cópias de segurança incrementais e completas.

**Teste recuperações parciais.** Uma recuperação parcial de uma parte da app pode fornecer garantias aos utilizadores de que todos não estão disponíveis. Esta parte do plano deve garantir que uma restauração parcial não tenha quaisquer efeitos colaterais, como um serviço de backup e restauro que interage com a app para desligá-lo graciosamente antes da cópia de segurança ser feita.

**Determine os instigadores de recuperação de desastres e atribua a responsabilidade.** Um plano de recuperação deve descrever quem, e que funções, pode iniciar ações de backup e recuperação para além do que pode ser apoiado e restaurado.

**Compare limiares de auto-cura com recuperação de desastres.** Determine as capacidades de auto-cura de uma aplicação para iniciar a recuperação automática e o tempo necessário para que a auto-cura de uma aplicação seja considerada uma falha ou sucesso. Determine os limiares para cada nuvem.

**Verifique a disponibilidade de funcionalidades de resiliência.** Determine a disponibilidade de funcionalidades e capacidades de resiliência para cada local. Se uma localização não fornecer as capacidades necessárias, considere integrar essa localização num serviço centralizado que forneça as funcionalidades de resiliência.

**Determinar os tempos de inatividade.** Determine o tempo de inatividade esperado devido à manutenção da app como um todo e como componentes de aplicações.

**Documente os procedimentos de resolução de problemas.** Defina procedimentos de resolução de problemas para a redistribuição de recursos e componentes de aplicações.

## <a name="manageability"></a>Capacidade de gestão

As considerações sobre como gere as suas aplicações híbridas são fundamentais para desenhar a sua arquitetura. Uma aplicação híbrida bem gerida fornece uma infraestrutura como código que permite a integração de um código de aplicação consistente num oleoduto de desenvolvimento comum. Ao implementar testes consistentes a nível do sistema e individuais de alterações na infraestrutura, pode garantir uma implementação integrada se as alterações passarem nos testes, permitindo a sua fusão no código fonte.

Para a discussão central deste pilar, veja [*DevOps*](/azure/architecture/framework/#devops) nos cinco pilares da excelência da arquitetura.

### <a name="manageability-checklist"></a>Lista de verificação de gestibilidade

**Implementar monitorização.** Utilize um sistema de monitorização centralizado de componentes de aplicações espalhados pelas nuvens para proporcionar uma visão agregada da sua saúde e desempenho. Este sistema inclui a monitorização tanto dos componentes da aplicação como das capacidades da plataforma relacionadas.

Determine as partes da aplicação que requerem monitorização.

**Coordenar políticas.** Cada local que uma aplicação híbrida abrange pode ter a sua própria política que abrange tipos de recursos permitidos, convenções de nomeação, etiquetas e outros critérios.

**Definir e usar papéis.** Como administrador de base de dados, é necessário determinar as permissões necessárias para diferentes personalidades (como um proprietário de uma aplicação, um administrador de base de dados e um utilizador final) que precisam de aceder aos recursos da aplicação. Estas permissões precisam de ser configuradas nos recursos e dentro da app. Um sistema de controlo de acesso baseado em funções (RBAC) permite-lhe definir estas permissões nos recursos da aplicação. Estes direitos de acesso são desafiantes quando todos os recursos são implantados numa única nuvem, mas requerem ainda mais atenção quando os recursos são espalhados pelas nuvens. Permissões em recursos definidos numa nuvem não se aplicam aos recursos definidos noutra nuvem.

**Utilize gasodutos CI/CD.** Um pipeline de Integração Contínua e Desenvolvimento Contínuo (CI/CD) pode fornecer um processo consistente de autoria e implementação de aplicações que se estendem por nuvens, e fornecer garantia de qualidade para a sua infraestrutura e app. Este oleoduto permite que a infraestrutura e a app sejam testadas numa nuvem e implantadas noutra nuvem. O pipeline permite-lhe mesmo implantar certos componentes da sua app híbrida numa nuvem e outros componentes para outra nuvem, formando essencialmente a base para a implementação de aplicações híbridas. Um sistema CI/CD é fundamental para lidar com as dependências que os componentes da aplicação têm uns para os outros durante a instalação, como a aplicação web que necessita de uma cadeia de ligação à base de dados.

**Gerir o ciclo de vida.** Como os recursos de uma aplicação híbrida podem abranger locais, a capacidade de gestão do ciclo de vida de cada localização precisa de ser agregada numa unidade de gestão de ciclo de vida único. Considere como são criados, atualizados e apagados.

**Examine estratégias de resolução de problemas.** Resolver problemas numa aplicação híbrida envolve mais componentes de aplicações do que a mesma app que está a ser executada numa única nuvem. Além da conectividade entre as nuvens, a aplicação está a funcionar em duas plataformas em vez de uma. Uma tarefa importante na resolução de problemas de aplicações híbridas é examinar a monitorização agregada de saúde e desempenho dos componentes da aplicação.

## <a name="security"></a>Segurança

A segurança é uma das principais considerações para qualquer app em nuvem e torna-se ainda mais crítica para aplicações híbridas em nuvem.

Para a discussão central deste pilar, veja [*a Segurança*](https://docs.microsoft.com/azure/architecture/guide/pillars#security) nos cinco pilares da excelência da arquitetura.

### <a name="security-checklist"></a>Lista de verificação de segurança

**Assuma a violação.** Se uma parte da app estiver comprometida, certifique-se de que existem soluções para minimizar a propagação da falha, não só dentro do mesmo local, mas também em locais.

**O monitor permitiu o acesso à rede.** Determine as políticas de acesso à rede para a aplicação, como por exemplo aceder apenas à aplicação a partir de uma sub-rede específica e permitir apenas que as portas e protocolos mínimos entre os componentes necessários para que a app funcione corretamente.

**Empregue uma autenticação robusta.** Um sistema de autenticação robusto é fundamental para a segurança da sua app. Considere utilizar um fornecedor de identidade federado que forneça capacidades únicas de entrada e utilize um ou mais dos seguintes esquemas: nome de utilizador e assinatura de palavra-passe, chaves públicas e privadas, autenticação de dois fatores ou multi-fatores e grupos de segurança fidedignos. Determine os recursos adequados para armazenar dados sensíveis e outros segredos para a autenticação de aplicações, além dos tipos de certificados e seus requisitos.

**Use encriptação.** Identifique quais as áreas da aplicação que utilizam encriptação, como por exemplo para armazenamento de dados ou comunicação do cliente e acesso.

**Utilize canais seguros.** Um canal seguro através das nuvens é fundamental para fornecer verificações de segurança e autenticação, proteção em tempo real, quarentena e outros serviços através das nuvens.

**Definir e usar papéis.** Implementar funções para configurações de recursos e acesso de identidade única através das nuvens. Determine os requisitos de controlo de acesso baseados em funções (RBAC) para a aplicação e os seus recursos de plataforma.

**Audite o seu sistema.** A monitorização do sistema pode registar e agregar dados tanto dos componentes da aplicação como das operações da plataforma de nuvem relacionadas.

## <a name="summary"></a>Resumo

Este artigo fornece uma lista de itens que são importantes a considerar durante a autoria e conceção das suas aplicações híbridas. Rever estes pilares antes de implementar a sua app impede-o de encontrar estas questões em interrupções de produção e potencialmente exigir que reveja o seu design.

Pode parecer uma tarefa demorada de antemão, mas facilmente obtém o seu retorno no investimento se desenhar a sua app com base nestes pilares.

## <a name="next-steps"></a>Passos seguintes

Para obter mais informações, consulte os seguintes recursos:

- [Nuvem híbrida](https://azure.microsoft.com/overview/hybrid-cloud/)
- [Aplicativos de nuvem híbrida](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Desenvolver modelo do Azure Resource Manager para consistência da cloud](https://aka.ms/consistency)
