---
title: Padrão kubernetes de alta disponibilidade usando Azure e Azure Stack Hub
description: Saiba como uma solução de cluster Kubernetes proporciona alta disponibilidade usando O Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281317"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Padrão de cluster kubernetes de alta disponibilidade

Este artigo descreve como arquiteto e operar uma infraestrutura altamente disponível baseada em Kubernetes usando o motor do Serviço Azure Kubernetes (AKS) no Azure Stack Hub. Este cenário é comum para organizações com cargas de trabalho críticas em ambientes altamente restritos e regulamentados. Organizações em domínios como finanças, defesa e governo.

## <a name="context-and-problem"></a>Contexto e problema

Muitas organizações estão a desenvolver soluções nativas em nuvem que alavancam serviços e tecnologias de última geração como kubernetes. Embora a Azure forneça centros de dados na maioria das regiões do mundo, por vezes existem casos e cenários de utilização de bordas onde as aplicações críticas empresariais devem funcionar num local específico. Considerações incluem:

- Sensibilidade à localização
- Latência entre os sistemas de aplicação e de instalação
- Conservação da largura de banda
- Conectividade
- Requisitos regulamentares ou legais

Azure, em combinação com o Azure Stack Hub, aborda a maioria destas preocupações. Um vasto conjunto de opções, decisões e considerações para uma implementação bem sucedida de Kubernetes em execução no Azure Stack Hub é descrito abaixo.

## <a name="solution"></a>Solução

Este padrão pressupõe que temos de lidar com um conjunto rigoroso de restrições. A aplicação deve ser executada no local e todos os dados pessoais não devem chegar aos serviços públicos em nuvem. A monitorização e outros dados não PII podem ser enviados para a Azure e ser processados lá. Serviços externos como um registo público de contentores ou outros podem ser acedidos, mas podem ser filtrados através de uma firewall ou servidor de procuração.

A aplicação da amostra aqui apresentada (com base no [Azure Kubernetes Service Workshop](/learn/modules/aks-workshop/)) foi concebida para utilizar soluções nativas de Kubernetes sempre que possível. Este design evita o bloqueio do fornecedor, em vez de utilizar serviços nativos da plataforma. Como exemplo, a aplicação utiliza uma base de dados mongoDB auto-hospedada em vez de um serviço PaaS ou serviço de base de dados externo.

[![Híbrido de padrão de aplicação](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

O diagrama anterior ilustra a arquitetura de aplicação da aplicação da amostra em execução em Kubernetes no Azure Stack Hub. A aplicação é composta por vários componentes, incluindo:

 1) Um cluster Kubernetes baseado em motor AKS no Azure Stack Hub.
 2) [cert-manager](https://www.jetstack.io/cert-manager/), que fornece um conjunto de ferramentas para a gestão de certificados em Kubernetes, usado para solicitar automaticamente certificados da Let's Encrypt.
 3) Um espaço de nomes Kubernetes que contém os componentes da aplicação para a extremidade frontal (ratings-web), api (ratings-api) e base de dados (ratings-mongodb).
 4) O Controlador Ingress que encaminha o tráfego HTTP/HTTPS para pontos finais dentro do cluster Kubernetes.

A aplicação da amostra é usada para ilustrar a arquitetura da aplicação. Todos os componentes são exemplos. A arquitetura contém apenas uma aplicação. Para obter uma elevada disponibilidade (HA), executaremos a implementação pelo menos duas vezes em duas instâncias diferentes do Azure Stack Hub - eles podem correr no mesmo local ou em dois (ou mais) sites diferentes:

![Arquitetura de Infraestruturas](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Serviços como o Registo de Contentores Azure, Azure Monitor, entre outros, estão hospedados fora do Azure Stack Hub em Azure ou no local. Este design híbrido protege a solução contra a interrupção de uma única instância do Azure Stack Hub.

## <a name="components"></a>Componentes

A arquitetura global consiste nos seguintes componentes:

**O Azure Stack Hub** é uma extensão do Azure que pode executar cargas de trabalho num ambiente no local, fornecendo serviços Azure no seu datacenter. Vá ao [Azure Stack Hub para](/azure-stack/operator/azure-stack-overview) saber mais.

**Azure Kubernetes Service Engine (Motor AKS)** é o motor por detrás da oferta de serviços gerida da Kubernetes, a Azure Kubernetes Service (AKS), que está disponível hoje em Azure. Para o Azure Stack Hub, o Motor AKS permite-nos implementar, escalar e atualizar clusters Kubernetes totalmente apresentados e auto-geridos utilizando as capacidades iaaS do Azure Stack Hub. Vá à [visão geral do motor AKS](https://github.com/Azure/aks-engine) para saber mais.

Vá a [Questões e Limitações Conhecidas](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) para saber mais sobre as diferenças entre o motor AKS em Azure e o motor AKS no Azure Stack Hub.

**A Azure Virtual Network (VNet)** é usada para fornecer a infraestrutura de rede em cada Azure Stack Hub para as Máquinas Virtuais (VMs) que hospedam a infraestrutura de cluster Kubernetes.

**Balanceador de Carga do Azure** é utilizado para o Ponto Final API de Kubernetes e para o Controlador Nginx Ingress. O equilibrador de carga encaminha o tráfego externo (por exemplo, Internet) para nós e VMs oferecendo um serviço específico.

**O Registo de Contentores Azure (ACR)** é utilizado para armazenar imagens privadas do Docker e gráficos helm, que são implantados no cluster. O motor AKS pode autenticar-se com o registo do contentor utilizando uma identidade AD AZure. Kubernetes não requer ACR. Pode utilizar outros registos de contentores, como o Docker Hub.

**Azure Repos** é um conjunto de ferramentas de controlo de versão que pode utilizar para gerir o seu código. Você também pode usar GitHub ou outros repositórios baseados em git. Vá ao [Azure Repos Overview](/azure/devops/repos/get-started/what-is-repos) para saber mais.

**A Azure Pipelines** faz parte da Azure DevOps Services e executa construções, testes e implementações automatizadas. Também pode utilizar soluções ci/CD de terceiros, como o Jenkins. Vá ao [Azure Pipeline Overview](/azure/devops/pipelines/get-started/what-is-azure-pipelines) para saber mais.

**O Azure Monitor** recolhe e armazena métricas e registos, incluindo métricas de plataforma para os serviços Azure na solução e telemetria de aplicações. Utilize estes dados para monitorizar a aplicação, configurar alertas e dashboards e realizar a análise de causa raiz de falhas. O Azure Monitor integra-se com Kubernetes para recolher métricas de controladores, nós e contentores, bem como troncos de contentores e troncos de nó master. Vá ao [Azure Monitor Overview](/azure/azure-monitor/overview) para saber mais.

**Gestor de Tráfego do Azure** é um equilibrador de carga baseado em DNS que lhe permite distribuir o tráfego da melhor forma para serviços em diferentes regiões do Azure ou implementações do Azure Stack Hub. Gestor de Tráfego também proporciona elevada disponibilidade e capacidade de resposta. Os pontos finais da aplicação devem estar acessíveis a partir do exterior. Existem outras soluções no local também disponíveis.

**O Controlador Kubernetes Ingress** expõe as rotas HTTP(S) a serviços num cluster kubernetes. Para este fim, pode utilizar-se o Nginx ou qualquer controlador de entrada adequado.

**Helm** é um gestor de pacotes para a implementação de Kubernetes, fornecendo uma maneira de agregar diferentes objetos Kubernetes como Implementações, Serviços, Segredos, em um único "gráfico". Pode publicar, implementar, controlar a gestão da versão e atualizar um objeto de gráfico. O registo do contentor Azure pode ser usado como repositório para armazenar gráficos de leme embalados.

## <a name="design-considerations"></a>Considerações de conceção

Este padrão segue algumas considerações de alto nível explicadas mais detalhadamente nas secções seguintes deste artigo:

- A aplicação utiliza soluções nativas de Kubernetes, para evitar o bloqueio do fornecedor.
- A aplicação utiliza uma arquitetura de microserviços.
- O Azure Stack Hub não precisa de entrada, mas permite a conectividade de saída da Internet.

Estas práticas recomendadas aplicar-se-ão também às cargas de trabalho e cenários do mundo real.

## <a name="scalability-considerations"></a>Considerações de escalabilidade

A escalabilidade é importante para fornecer aos utilizadores um acesso consistente, fiável e bem-realizado à aplicação.

O cenário da amostra cobre a escalabilidade em várias camadas da pilha de aplicações. Aqui está uma visão geral de alto nível das diferentes camadas:

| Nível de arquitetura | Afeta | Como posso fazê-lo? |
| --- | --- | ---
| Aplicação | Aplicação | Escala horizontal com base no número de cápsulas/réplicas/instâncias de contentores* |
| Cluster | Aglomerado de Kubernetes | Número de nós (entre 1 e 50), tamanhos VM-SKU e Piscinas de Nó (Motor AKS no Azure Stack Hub suporta atualmente apenas uma piscina de nó); utilizando o comando de escala do motor AKS (manual) |
| Infraestrutura | Azure Stack Hub | Número de nós, capacidade e unidades de escala dentro de uma implantação do Azure Stack Hub |

\* Utilizando o autoescala de pod horizontal de Kubernetes (HPA); escala métrica automatizada ou escala vertical dimensionando as instâncias do recipiente (cpu/memória).

**Azure Stack Hub (nível de infraestrutura)**

A infraestrutura Azure Stack Hub é a base desta implementação, porque o Azure Stack Hub funciona com hardware físico num datacenter. Ao selecionar o hardware do Hub, tem de fazer escolhas para CPU, densidade de memória, configuração de armazenamento e número de servidores. Para saber mais sobre a escalabilidade do Azure Stack Hub, confira os seguintes recursos:

- [Planeamento de capacidade para a visão geral do Azure Stack Hub](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Adicione os nóns de unidade de escala adicional no Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Aglomerado de Kubernetes (nível de cluster)**

O cluster Kubernetes em si é composto por, e é construído em cima de componentes IaaS Azure (Stack) incluindo computação, armazenamento e recursos de rede. As soluções Kubernetes envolvem os nódoas master e worker, que são implantados como VMs em Azure (e Azure Stack Hub).

- [Os nós de avião de controlo](/azure/aks/concepts-clusters-workloads#control-plane) (master) fornecem os serviços de Base Kubernetes e a orquestração das cargas de trabalho de aplicação.
- [Os nós dos trabalhadores](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (trabalhador) executam as suas cargas de trabalho de aplicação.

Ao selecionar tamanhos de VM para a implementação inicial, existem várias considerações:  

- **Custo** - Ao planear os nós dos seus trabalhadores, tenha em mente o custo global por VM que irá incorrer. Por exemplo, se as cargas de trabalho da sua aplicação requerem recursos limitados, deverá planear a implantação de VMs de menor dimensão. O Azure Stack Hub, tal como o Azure, é normalmente cobrado numa base de consumo, por isso o dimensionamento adequado dos VMs para os papéis de Kubernetes é crucial para otimizar os custos de consumo. 

- **Escalaability** - A escalabilidade do cluster é conseguida escalando e superando o número de nós mestres e trabalhadores, ou adicionando piscinas de nó adicionais (não disponível hoje no Azure Stack Hub). A escala do cluster pode ser feita com base em dados de desempenho, recolhidos utilizando Informações de contentores (Azure Monitor + Log Analytics). 

    Se a sua aplicação precisar de mais (ou menos) recursos, pode escalar (ou entrar) os seus nós atuais horizontalmente (entre 1 e 50 nós). Se precisar de mais de 50 nós, pode criar um cluster adicional numa subscrição separada. Não é possível escalar os VMs reais verticalmente para outro tamanho VM sem recolocar o cluster.

    O dimensionamento é feito manualmente usando o VM do ajudante do motor AKS que foi usado para implantar o cluster Kubernetes inicialmente. Para mais informações, consulte [os clusters Scaling Kubernetes](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Quotas** - Considere as [quotas](/azure-stack/operator/azure-stack-quota-types) que configuraste ao planear uma implantação AKS no seu Azure Stack Hub. Certifique-se de que cada [subscrição](/azure-stack/operator/service-plan-offer-subscription-overview) tem os planos adequados e as quotas configuradas. A subscrição terá de acomodar a quantidade de computação, armazenamento e outros serviços necessários para os seus clusters à medida que se escalonam.

- **Cargas de trabalho de aplicação** - Consulte os [conceitos de Clusters e cargas de trabalho](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) nos conceitos fundamentais de Kubernetes para o documento de serviço Azure Kubernetes. Este artigo irá ajudá-lo a estender o tamanho vm adequado com base nas necessidades de cálculo e memória da sua aplicação.  

**Aplicação (nível de aplicação)**

Na camada de aplicação, utilizamos Kubernetes [Horizontal Pod Autoscaler (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). O HPA pode aumentar ou diminuir o número de réplicas (Pod/Container Instances) na nossa implantação com base em diferentes métricas (como a utilização do CPU).

Outra opção é escalar os casos de contentores verticalmente. Isto pode ser conseguido alterando a quantidade de CPU e Memória solicitadas e disponíveis para uma implementação específica. Consulte [a Gestão de Recursos para Contentores](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) em kubernetes.io para saber mais.

## <a name="networking-and-connectivity-considerations"></a>Considerações de networking e conectividade

O networking e a conectividade também afetam as três camadas mencionadas anteriormente para Kubernetes no Azure Stack Hub. A tabela a seguir mostra as camadas e quais os serviços que contêm:

| Camada | Afeta | What? |
| --- | --- | ---
| Aplicação | Aplicação | Como é que a aplicação está acessível? Será exposto à Internet? |
| Cluster | Aglomerado de Kubernetes | Kubernetes API, AKS Engine VM, Pull container images (egress), Envio de dados de monitorização e telemetria (saída) |
| Infraestrutura | Azure Stack Hub | Acessibilidade dos pontos finais de gestão do Azure Stack Hub, como o portal e os pontos finais do Azure Resource Manager. |

**Aplicação**

Para a camada de aplicação, a consideração mais importante é se a aplicação está exposta e acessível a partir da Internet. Do ponto de vista de Kubernetes, a acessibilidade à Internet significa expor uma implementação ou pod usando um Serviço Kubernetes ou um Controlador ingress.

> [!NOTE]
> Recomendamos a utilização de controladores Ingress para expor os Serviços Kubernetes, uma vez que o número de IPs públicos frontend no Azure Stack Hub está limitado a 5. Este design também limita o número de Serviços Kubernetes (com o tipo LoadBalancer) a 5, que será muito pequeno para muitas implementações. Aceda à documentação do [Motor AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) para saber mais.

Expor uma aplicação utilizando um IP público através de um Balancer de Carga ou de um Controlador de Entradas não significa que a aplicação esteja agora acessível através da Internet. É possível que o Azure Stack Hub tenha um endereço IP público que só seja visível na intranet local - nem todos os IPs públicos estão verdadeiramente virados para a Internet.

O bloco anterior considera o tráfego ingresso para a aplicação. Outro tópico que deve ser considerado para uma implementação bem sucedida de Kubernetes é o tráfego de saída/saída. Aqui estão alguns casos de uso que requerem tráfego de saída:

- Imagens de contentores armazenados no Registo de Contentores DockerHub ou Azure
- Recuperação de gráficos de leme
- Emissão de Dados Informações de Aplicação (ou outros dados de monitorização)

Alguns ambientes empresariais podem exigir a utilização de servidores proxy _transparentes_ ou _não transparentes._ Estes servidores requerem configuração específica em vários componentes do nosso cluster. A documentação do Motor AKS contém vários detalhes sobre como acomodar proxies de rede. Para mais detalhes, consulte [o Motor AKS e servidores proxy](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Por último, o tráfego de aglomerados cruzados deve fluir entre as instâncias do Azure Stack Hub. A distribuição da amostra consiste em clusters individuais de Kubernetes que executam em instâncias individuais do Azure Stack Hub. O trânsito entre eles, como o tráfego de replicação entre duas bases de dados, é de "tráfego externo". O tráfego externo deve ser encaminhado através de um endereço IP público do Site-to-Site ou do Azure Stack Hub para ligar Kubernetes em duas instâncias do Azure Stack Hub:

![tráfego inter e intra cluster](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

O cluster Kubernetes não precisa necessariamente de ser acessível através da Internet. A parte relevante é a API kubernetes usada para operar um cluster, por exemplo, utilizando `kubectl` . O ponto final da API kubernetes deve estar acessível a todos os que operam o cluster ou implementam aplicações e serviços em cima dele. Este tópico é abordado mais detalhadamente a partir de uma perspetiva de DevOps na secção [de considerações de Implementação (CI/CD)](#deployment-cicd-considerations) abaixo.

No nível do cluster, há também algumas considerações em torno do tráfego de saídas:

- Atualizações de nó (para Ubuntu)
- Dados de monitorização (enviados à Azure LogAnalytics)
- Outros agentes que exigem tráfego de saída (específico para o ambiente de cada desdobramento)

Antes de implementar o seu cluster Kubernetes utilizando o Motor AKS, planeie o design final de rede. Em vez de criar uma Rede Virtual dedicada, pode ser mais eficiente implantar um cluster numa rede existente. Por exemplo, pode aproveitar uma ligação VPN local-a-local já configurada no seu ambiente Azure Stack Hub.

**Infraestrutura**  

A infraestrutura refere-se ao acesso aos pontos finais de gestão do Azure Stack Hub. Os pontos finais incluem os portais de arrendamento e administração, e os pontos finais do Administrador de Recursos Azure e inquilinos. Estes pontos finais são necessários para operar o Azure Stack Hub e os seus serviços principais.

## <a name="data-and-storage-considerations"></a>Considerações de dados e armazenamento

Duas instâncias da nossa aplicação serão implementadas em dois clusters individuais de Kubernetes, através de duas instâncias do Azure Stack Hub. Este design vai exigir que consideremos como replicar e sincronizar dados entre eles.

Com o Azure, temos a capacidade incorporada de replicar o armazenamento em várias regiões e zonas dentro da nuvem. Atualmente com o Azure Stack Hub não existem formas nativas de replicar o armazenamento em duas instâncias diferentes do Azure Stack Hub - formam duas nuvens independentes sem forma abrangente de as gerir como um conjunto. O planeamento da resiliência das aplicações que atravessam o Azure Stack Hub obriga-o a considerar esta independência no design e implementações da sua aplicação.

Na maioria dos casos, a replicação de armazenamento não será necessária para uma aplicação resiliente e altamente disponível implantada em AKS. Mas deve considerar o armazenamento independente por exemplo do Azure Stack Hub no design da sua aplicação. Se este design for uma preocupação ou um bloqueio de estrada para implementar a solução no Azure Stack Hub, existem possíveis soluções da Microsoft Partners que fornecem anexos de armazenamento. Armazenamento anexos fornecem uma solução de replicação de armazenamento em vários Hubs Azure Stack e Azure. Para mais informações, consulte as [soluções Partner.](#partner-solutions)

Na nossa arquitetura, estas camadas foram consideradas:

**Configuração**

A configuração inclui a configuração do Azure Stack Hub, do Motor AKS e do próprio cluster Kubernetes. A configuração deve ser automatizada tanto quanto possível, e armazenada como Infra-estrutura-como-Código num sistema de controlo de versão baseado em Git, como Azure DevOps ou GitHub. Estas definições não podem ser facilmente sincronizadas em várias implementações. Por isso, recomendamos armazenar e aplicar a configuração a partir do exterior e utilizar o gasoduto DevOps.

**Aplicação**

A aplicação deve ser armazenada num repositório baseado em Git. Sempre que há uma nova implementação, alterações na aplicação, ou recuperação de desastres, pode ser facilmente implantada usando Azure Pipelines.

**Dados**

Os dados são a consideração mais importante na maioria dos desenhos de aplicações. Os dados da aplicação devem permanecer sincronizados entre as diferentes instâncias da aplicação. Os dados também precisam de uma estratégia de backup e recuperação de desastres se houver uma paragem.

Alcançar este design depende muito das escolhas tecnológicas. Aqui estão alguns exemplos de solução para implementar uma base de dados de uma forma altamente disponível no Azure Stack Hub:

- [Implementar um grupo de disponibilidade SQL Server 2016 para o Azure e o Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Implemente uma solução MongoDB altamente disponível para O Azure e Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Considerações ao trabalhar com dados em vários locais é uma consideração ainda mais complexa para uma solução altamente disponível e resiliente. Considere:

- Latência e conectividade de rede entre Azure Stack Hubs.
- Disponibilidade de identidades para serviços e permissões. Cada exemplo do Azure Stack Hub integra-se com um diretório externo. Durante a implantação, opta por utilizar os serviços de Azure Ative Directory (Azure AD) ou da Federação de Diretórios Ativos (ADFS). Como tal, existe o potencial de usar uma única identidade que pode interagir com vários casos independentes do Azure Stack Hub.

## <a name="business-continuity-and-disaster-recovery"></a>Continuidade de negócio e recuperação após desastre

A continuidade do negócio e a recuperação de desastres (BCDR) é um tema importante tanto no Azure Stack Hub como no Azure. A principal diferença é que no Azure Stack Hub, o operador deve gerir todo o processo BCDR. Em Azure, partes do BCDR são geridas automaticamente pela Microsoft.

O BCDR afeta as mesmas áreas mencionadas na secção anterior [Dados e considerações de armazenamento:](#data-and-storage-considerations)

- Infraestrutura / Configuração
- Disponibilidade de candidaturas
- Dados da Aplicação

E como mencionado na secção anterior, estas áreas são da responsabilidade do operador Azure Stack Hub e podem variar entre organizações. Planeie o BCDR de acordo com as suas ferramentas e processos disponíveis.

**Infraestrutura e configuração**

Esta secção abrange a infraestrutura física e lógica e a configuração do Azure Stack Hub. Abrange ações na administração e nos espaços de arrendamento.

O operador do Azure Stack Hub (ou administrador) é responsável pela manutenção das instâncias Azure Stack Hub. Incluindo componentes como a rede, armazenamento, identidade e outros tópicos que estão fora do âmbito deste artigo. Para saber mais sobre as especificidades das operações do Azure Stack Hub, consulte os seguintes recursos:

- [Recuperar dados no Azure Stack Hub com o Serviço de Backup de Infraestruturas](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Ativar a cópia de segurança para O Azure Stack Hub a partir do portal do administrador](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Recover from catastrophic data loss](/azure-stack/operator/azure-stack-backup-recover-data) (Recuperar de uma perda de dados catastrófica)
- [Infrastructure Backup Service best practices](/azure-stack/operator/azure-stack-backup-best-practices) (Melhores práticas do Infrastructure Backup Service)

Azure Stack Hub é a plataforma e tecido em que serão implementadas aplicações Kubernetes. O proprietário da aplicação Kubernetes será um utilizador do Azure Stack Hub, com acesso concedido para implementar a infraestrutura de aplicação necessária para a solução. Neste caso, a infraestrutura de aplicações significa o cluster Kubernetes, implantado com motor AKS e os serviços circundantes. Estes componentes serão implantados no Azure Stack Hub, limitados por uma oferta do Azure Stack Hub. Certifique-se de que a oferta aceite pelo proprietário da aplicação Kubernetes tem capacidade suficiente (expressa nas quotas Azure Stack Hub) para implementar toda a solução. Como recomendado na secção anterior, a implementação da aplicação deve ser automatizada utilizando infraestruturas como código e oleodutos de implantação como os gasodutos Azure DevOps Azure.

Para obter mais informações sobre ofertas e quotas do Azure Stack Hub, consulte [os serviços, planos, ofertas e subscrições do Azure Stack Hub](/azure-stack/operator/service-plan-offer-subscription-overview)

É importante guardar e armazenar de forma segura a configuração do Motor AKS, incluindo as suas saídas. Estes ficheiros contêm informações confidenciais utilizadas para aceder ao cluster Kubernetes, pelo que devem ser protegidos contra a exposição a não administradores.

**Disponibilidade de candidaturas**

A aplicação não deve basear-se em cópias de segurança de uma instância implantada. Como prática padrão, reimplante a aplicação completamente seguindo os padrões infra-como-código. Por exemplo, reenfectuar utilizando gasodutos Azure DevOps Azure. O procedimento BCDR deve envolver a reafectação do pedido para o mesmo ou outro cluster Kubernetes.

**Dados da aplicação**

Os dados da aplicação são a parte crítica para minimizar a perda de dados. Na secção anterior, foram descritas técnicas para replicar e sincronizar dados entre duas (ou mais) instâncias da aplicação. Dependendo da infraestrutura de base de dados (MySQL, MongoDB, MSSQL ou outros) utilizada para armazenar os dados, haverá diferentes técnicas de disponibilidade de base de dados e técnicas de backup disponíveis para escolher.

As formas recomendadas de alcançar a integridade são usar:
- Uma solução de reserva nativa para a base de dados específica.
- Uma solução de backup que suporta oficialmente o backup e a recuperação do tipo de base de dados utilizado pela sua aplicação.

> [!IMPORTANT]
> Não guarde os seus dados de backup na mesma instância do Azure Stack Hub onde os dados da sua aplicação residem. Uma paragem completa da instância Azure Stack Hub também comprometeria as suas cópias de segurança.

## <a name="availability-considerations"></a>Considerações de disponibilidade

Kubernetes no Azure Stack Hub implantado através do Motor AKS não é um serviço gerido. É uma implementação automatizada e configuração de um cluster Kubernetes usando Azure Infrastructure-as-a-Service (IaaS). Como tal, proporciona a mesma disponibilidade que a infraestrutura subjacente.

A infraestrutura Azure Stack Hub já é resistente a falhas, e fornece capacidades como Conjuntos de Disponibilidade para distribuir componentes em vários [domínios de falha e atualização](/azure-stack/user/azure-stack-vm-considerations#high-availability). Mas a tecnologia subjacente (clustering failover) ainda incorre em algum tempo de inatividade para VMs em um servidor físico impactado, se houver uma falha de hardware.

É uma boa prática implementar o seu cluster Kubernetes de produção, bem como a carga de trabalho para dois (ou mais) clusters. Estes clusters devem ser hospedados em diferentes locais ou datacenters, e usar tecnologias como Gestor de Tráfego do Azure para encaminhar os utilizadores com base no tempo de resposta do cluster ou com base na geografia.

![Utilizar Gestor de Tráfego para controlar os fluxos de tráfego](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Os clientes que têm um único cluster Kubernetes normalmente conectam-se ao nome IP ou DNS de serviço de uma determinada aplicação. Numa implementação multi-cluster, os clientes devem ligar-se a um nome DNS Gestor de Tráfego que aponta para os serviços/entradas em cada cluster Kubernetes.

![Usando Gestor de Tráfego para o cluster de rotas para o aglomerado no local](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Este padrão é também uma [melhor prática para clusters AKS (geridos) em Azure.](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)

O cluster Kubernetes em si, implantado através do motor AKS, deve ser composto por pelo menos três nós mestres e dois nós operários.

## <a name="identity-and-security-considerations"></a>Considerações de identidade e segurança

Identidade e segurança são temas importantes. Especialmente quando a solução abrange instâncias independentes do Azure Stack Hub. Kubernetes e Azure (incluindo O Azure Stack Hub) ambos têm mecanismos distintos para o controlo de acesso baseado em funções (RBAC):

- O Azure RBAC controla o acesso aos recursos em Azure (e Azure Stack Hub), incluindo a capacidade de criar novos recursos Azure. As permissões podem ser atribuídas a utilizadores, grupos ou principais serviços. (Um principal de serviço é uma identidade de segurança utilizada por aplicações.)
- Kubernetes RBAC controla permissões para a API de Kubernetes. Por exemplo, criar pods e cápsulas de listagem são ações que podem ser autorizadas (ou negadas) a um utilizador através do RBAC. Para atribuir permissões kubernetes aos utilizadores, cria funções e encadernações de papéis.

**Identidade do Azure Stack Hub e RBAC**

O Azure Stack Hub oferece duas opções de fornecedor de identidade. O fornecedor que utiliza depende do ambiente e do funcionamento num ambiente ligado ou desligado:

- Azure AD - só pode ser usado num ambiente conectado.
- ADFS para uma floresta tradicional de Diretório Ativo - pode ser usado em um ambiente conectado ou desligado.

O fornecedor de identidade gere utilizadores e grupos, incluindo a autenticação e autorização de acesso aos recursos. O acesso pode ser concedido aos recursos do Azure Stack Hub, como subscrições, grupos de recursos e recursos individuais, como VMs ou equilibradores de carga. Para ter um modelo de acesso consistente, deve considerar a utilização dos mesmos grupos (diretos ou aninhados) para todos os Azure Stack Hubs. Aqui está um exemplo de configuração:

![grupos aad aninhados com hub pilha azul](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

O exemplo contém um grupo dedicado (utilizando AAD ou ADFS) para um propósito específico. Por exemplo, para fornecer permissões de contribuinte para o Grupo de Recursos que contém a nossa infraestrutura de cluster Kubernetes numa instância específica do Azure Stack Hub (aqui "Seattle K8s Cluster Contributor"). Estes grupos são então aninhados num grupo global que contém os "subgrupos" para cada Azure Stack Hub.

O nosso utilizador da amostra terá agora permissões de "Contribuinte" para ambos os Grupos de Recursos que contêm todo o conjunto de recursos de infraestrutura kubernetes. O utilizador terá acesso a recursos em ambos os casos do Azure Stack Hub, porque as instâncias partilham o mesmo fornecedor de identidade.

> [!IMPORTANT]
> Estas permissões afetam apenas o Azure Stack Hub e alguns dos recursos implantados em cima dele. Um utilizador que tenha este nível de acesso pode causar muitos danos, mas não consegue aceder aos VMs de Kubernetes nem à API de Kubernetes sem acesso adicional à implementação de Kubernetes.

**Identidade de Kubernetes e RBAC**

Um cluster Kubernetes, por padrão, não usa o mesmo Fornecedor de Identidade que o Azure Stack Hub. Os VMs que acolhem o cluster Kubernetes, o mestre e os nós dos trabalhadores, utilizam a Chave SSH que é especificada durante a implantação do cluster. Esta chave SSH é necessária para ligar a estes nós utilizando SSH.

A API de Kubernetes (por exemplo, acedida através da `kubectl` utilização) também está protegida por contas de serviço, incluindo uma conta de serviço "cluster administrador" por defeito. As credenciais desta conta de serviço são inicialmente `.kube/config` armazenadas no ficheiro nos seus nós principais Kubernetes.

**Segredos de gestão e credenciais de aplicação**

Para armazenar segredos como cadeias de ligação ou credenciais de base de dados existem várias opções, incluindo:

- Azure Key Vault
- Segredos de Kubernetes
- Soluções de 3ª Parte como o HashiCorp Vault (em execução em Kubernetes)

Não guarde segredos ou credenciais em texto simples nos seus ficheiros de configuração, código de aplicação ou dentro de scripts. E não os guarde num sistema de controlo de versão. Em vez disso, a automatização de implantação deve recuperar os segredos conforme necessário.

## <a name="patch-and-update"></a>Patch e atualização

O processo **Patch and Update (PNU)** no Serviço Azure Kubernetes é parcialmente automatizado. As atualizações da versão Kubernetes são ativadas manualmente, enquanto as atualizações de segurança são aplicadas automaticamente. Estas atualizações podem incluir correções de segurança do SISTEMA ou atualizações de kernel. A AKS não reinicia automaticamente estes nós Linux para completar o processo de atualização. 

O processo PNU para um cluster Kubernetes implantado usando o motor AKS no Azure Stack Hub não é gerido, e é da responsabilidade do operador de cluster. 

O Motor AKS ajuda com as duas tarefas mais importantes:

- [Upgrade para uma versão de imagem mais recente de Kubernetes e base OS](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Atualizar apenas a imagem base do SO](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

As imagens de OS de base mais recentes contêm as mais recentes correções de segurança do SO e atualizações de kernel. 

O mecanismo [de atualização não acompanhado](https://wiki.debian.org/UnattendedUpgrades) instala automaticamente atualizações de segurança que são lançadas antes de uma nova versão base de imagem do SO estar disponível no Azure Stack Hub Marketplace. A atualização não acompanhada é ativada por padrão e instala atualizações de segurança automaticamente, mas não reinicia os nós do cluster Kubernetes. Reiniciar os nós pode ser automatizado utilizando a [ bota **RE** **D** Aemon (kured))](/azure/aks/node-updates-kured). Relógios kured para nós Linux que requerem um reboot, em seguida, lidar automaticamente com o reagendamento de cápsulas de corrida e processo de reinicialização de nó.

## <a name="deployment-cicd-considerations"></a>Considerações de implantação (CI/CD)

Azure e Azure Stack Hub expõem as mesmas APIs do Gestor de Recursos Azure. Estas APIs são endereçadas como qualquer outra nuvem Azure (Azure, Azure China 21Vianet, Governo Azure). Pode haver diferenças nas versões API entre nuvens, e o Azure Stack Hub fornece apenas um subconjunto de serviços. O ponto final de gestão URI também é diferente para cada nuvem, e cada instância de Azure Stack Hub.

Além das diferenças subtis mencionadas, as APIs do Gestor de Recursos Azure fornecem uma forma consistente de interagir com o Azure e o Azure Stack Hub. O mesmo conjunto de ferramentas pode ser usado aqui como seria usado com qualquer outra nuvem Azure. Você pode usar Azure DevOps, ferramentas como Jenkins, ou PowerShell, para implementar e orquestrar serviços para Azure Stack Hub.

**Considerações**

Uma das grandes diferenças no que diz respeito às implementações do Azure Stack Hub é a questão da acessibilidade à Internet. A acessibilidade à Internet determina se seleciona um agente de construção hospedado na Microsoft ou um agente de construção auto-hospedado para os seus trabalhos de CI/CD.

Um agente auto-hospedado pode funcionar em cima do Azure Stack Hub (como um IaaS VM) ou numa sub-rede de rede que pode aceder ao Azure Stack Hub. Vá aos [agentes da Azure Pipelines](/azure/devops/pipelines/agents/agents) para saber mais sobre as diferenças.

A imagem a seguir ajuda-o a decidir se precisa de um agente de construção auto-hospedado ou hospedado na Microsoft:

![Agentes de construção auto-hospedados Sim ou Não](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Os pontos finais de gestão do Azure Stack Hub estão acessíveis via Internet?
  - Sim: Podemos usar Azure Pipelines com agentes hospedados pela Microsoft para ligar ao Azure Stack Hub.
  - Não: Precisamos de agentes auto-hospedados que possam ligar-se aos pontos finais de gestão do Azure Stack Hub.
- O nosso cluster Kubernetes está acessível através da Internet?
  - Sim: Podemos usar Azure Pipelines com agentes hospedados pela Microsoft para interagir diretamente com o ponto final da API da Kubernetes.
  - Não: Precisamos de agentes auto-hospedados que possam ligar-se ao ponto final da API do cluster Kubernetes.

Em cenários em que os pontos finais de gestão do Azure Stack Hub e da API de Kubernetes estão acessíveis através da internet, a implementação pode utilizar um agente hospedado pela Microsoft. Esta implementação resultará numa arquitetura de aplicações da seguinte forma:

[![Visão geral da arquitetura pública](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Se o Gestor de Recursos Azure, a API de Kubernetes, ou ambos não estiverem diretamente acessíveis através da Internet, podemos aproveitar um agente de construção auto-hospedado para executar os passos do pipeline. Este design necessita de menos conectividade, e pode ser implementado apenas com conectividade de rede no local para pontos finais do Azure Resource Manager e da API de Kubernetes:

[![Visão geral da arquitetura on-prem](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **E cenários desligados?** Em cenários em que o Azure Stack Hub ou o Kubernetes ou ambos não têm pontos finais de gestão virados para a Internet, ainda é possível utilizar o Azure DevOps para as suas implementações. Você pode usar um Agent Pool auto-hospedado (que é um Agente DevOps que corre no local ou no próprio Azure Stack Hub) ou um Azure DevOps Server completamente auto-hospedado no local. O agente auto-alojado necessita apenas de conectividade de saída HTTPS (TCP/443) na Internet.

O padrão pode usar um cluster Kubernetes (implantado e orquestrado com motor AKS) em cada instância do Azure Stack Hub. Inclui uma aplicação composta por um frontend, um serviço de backend de nível médio (por exemplo, MongoDB), e um Controlador ingress baseado em Nginx. Em vez de utilizar uma base de dados hospedada no cluster K8s, pode aproveitar as "lojas de dados externas". As opções de base de dados incluem MySQL, SQL Server ou qualquer tipo de base de dados hospedada fora do Azure Stack Hub ou em IaaS. Configurações como esta não estão no âmbito aqui.

## <a name="partner-solutions"></a>Soluções de parceiros

Existem soluções Microsoft Partner que podem alargar as capacidades do Azure Stack Hub. Estas soluções foram consideradas úteis nas implementações de aplicações em execução em clusters Kubernetes.  

## <a name="storage-and-data-solutions"></a>Armazenamento e soluções de dados

Como descrito em [considerações de Dados e armazenamento,](#data-and-storage-considerations)a Azure Stack Hub atualmente não tem uma solução nativa para replicar armazenamento em várias instâncias. Ao contrário do Azure, a capacidade de replicar o armazenamento em várias regiões não existe. Em Azure Stack Hub, cada instância é a sua própria nuvem distinta. No entanto, existem soluções disponíveis a partir de Microsoft Partners que permitem a replicação de armazenamento através do Azure Stack Hubs e do Azure. 

**ESCALIDADE**

[A Scality](https://www.scality.com/) oferece armazenamento em escala web que alimenta empresas digitais desde 2009. O Scality RING, o nosso armazenamento definido por software, transforma os servidores de mercadoria x86 num conjunto de armazenamento ilimitado para qualquer tipo de dados – ficheiro e objeto – à escala de petabyte.

**CLOUDIAN**

[O Cloudian](https://www.cloudian.com/) simplifica o armazenamento da empresa com armazenamento escalável ilimitado que consolida conjuntos de dados maciços para um ambiente único e facilmente gerido.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre conceitos introduzidos neste artigo:

- [Escalas cruzadas](pattern-cross-cloud-scale.md) e [padrões de aplicativos geo-distribuídos](pattern-geo-distributed.md) no Azure Stack Hub.
- [Microserviços arquitetura em Azure Kubernetes Service (AKS)](/azure/architecture/reference-architectures/microservices/aks).

Quando estiver pronto para testar o exemplo da solução, continue com o guia de implantação do [cluster Kubernetes](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)de alta disponibilidade. O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.