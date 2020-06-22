---
title: Escalamento de nuvem cruzada (dados no local) padrão no Azure Stack Hub
description: Saiba como construir uma aplicação cross-cloud escalável que utiliza dados on-prem em Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911125"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Padrão de escala de nuvem cruzada (dados no local)

Saiba como construir uma aplicação híbrida que se estende por Azure e Azure Stack Hub. Este padrão também mostra como usar uma única fonte de dados no local para o cumprimento.

## <a name="context-and-problem"></a>Contexto e problema

Muitas organizações recolhem e armazenam quantidades massivas de dados sensíveis do cliente. Frequentemente são impedidos de armazenar dados sensíveis na nuvem pública por causa de regulamentos corporativos ou política do governo. Essas organizações também querem aproveitar a escalabilidade da nuvem pública. A nuvem pública pode lidar com picos sazonais no tráfego, permitindo que os clientes paguem exatamente o hardware de que precisam, quando precisam.

## <a name="solution"></a>Solução

A solução tira partido dos benefícios de conformidade da nuvem privada, combinando-os com a escalabilidade da nuvem pública. A nuvem híbrida Azure e Azure Stack Hub proporcionam uma experiência consistente para os desenvolvedores. Esta consistência permite-lhes aplicar as suas competências tanto em ambientes de nuvem pública como de ambientes no local.

O guia de implementação da solução permite-lhe implementar uma aplicação web idêntica para uma nuvem pública e privada. Também pode aceder a uma rede de encaminhamento não internet hospedada na nuvem privada. As aplicações web são monitorizadas para a carga. Após um aumento significativo do tráfego, um programa manipula os registos dns para redirecionar o tráfego para a nuvem pública. Quando o tráfego já não é significativo, os registos dns são atualizados para direcionar o tráfego de volta para a nuvem privada.

[![Escala de nuvem cruzada com padrão de dados on-prem](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Componentes

Esta solução utiliza os seguintes componentes:

| Camada | Componente | Description |
|----------|-----------|-------------|
| Azure | Serviço de Aplicações do Azure | [O Azure App Service](/azure/app-service/) permite-lhe construir e hospedar aplicações web, aplicações API RESTful e Funções Azure. Tudo na linguagem de programação à sua escolha, sem gerir infraestruturas. |
| | Rede Virtual do Azure| [A Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) é o bloco de construção fundamental para redes privadas em Azure. O VNet permite que vários tipos de recursos Azure, como máquinas virtuais (VM), comuniquem-se de forma segura entre si, a internet e as redes no local. A solução demonstra igualmente a utilização de componentes adicionais de rede:<br>- aplicativos e sub-redes gateway.<br>- uma porta de entrada de rede local.<br>- um gateway de rede virtual, que funciona como uma ligação de gateway VPN local.<br>- um endereço IP público.<br>- uma ligação VPN ponto-a-local.<br>- Azure DNS para hospedar domínios DNS e fornecer resolução de nome. |
| | Traffic Manager do Azure | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) é um equilibrador de carga baseado em DNS. Permite controlar a distribuição do tráfego de utilizadores para pontos finais de serviço em diferentes datacenters. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) é um serviço extensível de Gestão de Desempenho de Aplicações para desenvolvedores web que construem e gerem aplicações em várias plataformas.|
| | Funções do Azure | [As Funções Azure](/azure/azure-functions/) permitem executar o seu código num ambiente sem servidor sem ter de criar primeiro um VM ou publicar uma aplicação web. |
| | Dimensionamento Automático do Azure | [Autoscale](/azure/azure-monitor/platform/autoscale-overview) é uma funcionalidade incorporada de Serviços cloud, VMs e aplicativos web. A funcionalidade permite que as aplicações executem o seu melhor quando a procura muda. As aplicações ajustar-se-ão aos picos de tráfego, notificando-o quando as métricas mudarem e escalarem conforme necessário. |
| Azure Stack Hub | IaaS Compute | O Azure Stack Hub permite-lhe utilizar o mesmo modelo de aplicação, portal de self-service e APIs ativados pelo Azure. O Azure Stack Hub IaaS permite uma ampla gama de tecnologias de código aberto para implementações consistentes em nuvem híbrida. O exemplo da solução utiliza um VM do Servidor Windows para o SQL Server, por exemplo.|
| | Serviço de Aplicações do Azure | Tal como a aplicação web Azure, a solução utiliza o [Azure App Service no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) para hospedar a aplicação web. |
| | Redes | A Rede Virtual Azure Stack Hub funciona exatamente como a Rede Virtual Azure. Utiliza muitos dos mesmos componentes de rede, incluindo os anfitriões personalizados.
| Serviços de DevOps do Azure | Inscrever-se | Configurar rapidamente a integração contínua para construção, teste e implantação. Para mais informações, consulte [Inscrever-se, iniciar súm na Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Pipelines do Azure | Utilize [gasodutos Azure](/azure/devops/pipelines/agents/agents?view=azure-devops) para integração contínua/entrega contínua. A Azure Pipelines permite-lhe gerir agentes e definições de construção e libertação hospedados. |
| | Repositório de código | Aproveite vários repositórios de código para simplificar o seu oleoduto de desenvolvimento. Utilize repositórios de código existentes em GitHub, Bitbucket, Dropbox, OneDrive e Azure Repos. |

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esta solução:

### <a name="scalability"></a>Escalabilidade

O Azure e o Azure Stack Hub são exclusivamente adequados para suportar as necessidades do negócio distribuído globalmente.

#### <a name="hybrid-cloud-without-the-hassle"></a>Nuvem híbrida sem o ausísmos

A Microsoft oferece uma integração incomparável de ativos no local com o Azure Stack Hub e o Azure numa solução unificada. Esta integração elimina o incómodo de gerir soluções de múltiplos pontos e uma mistura de fornecedores de nuvem. Com a escala de nuvens cruzadas, o poder de Azure está a poucos cliques de distância. Basta ligar o seu Azure Stack Hub ao Azure com o rebentamento da nuvem e os seus dados e aplicações estarão disponíveis no Azure quando necessário.

- Elimine a necessidade de construir e manter um local de DR secundário.
- Poupe tempo e dinheiro eliminando a cópia de segurança da fita e aloja até 99 anos de dados de backup em Azure.
- Migrar facilmente as cargas de trabalho de Hiper-V, Física (em pré-visualização) e VMware (na pré-visualização) para Azure para alavancar a economia e a elasticidade da nuvem.
- Executar relatórios intensivos de cálculo ou análises sobre uma cópia replicada do seu ativo no local em Azure sem afetar as cargas de trabalho de produção.
- Explodiu na nuvem e correu no local cargas de trabalho em Azure, com modelos de computação maiores quando necessário. O híbrido dá-te o poder de que precisas, quando precisas.
- Crie ambientes de desenvolvimento de vários níveis em Azure com alguns cliques – até mesmo replicar dados de produção ao vivo para o seu ambiente dev/teste para mantê-lo em quase sincronização em tempo real.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Economia de escala cruzada com Azure Stack Hub

A principal vantagem para o rebentamento de nuvens é a poupança económica. Só se paga pelos recursos adicionais quando há uma procura por esses recursos. Não há mais gastos com capacidadeexuais desnecessárias ou a tentar prever picos de procura e flutuações.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Reduzir cargas de alta procura na nuvem

A escala de nuvens cruzadas pode ser usada para suportar cargas de processamento de ombros. A carga é distribuída através da mudança de aplicações básicas para a nuvem pública, libertando recursos locais para aplicações críticas ao negócio. Uma aplicação pode ser aplicada na nuvem privada, e depois explodir para a nuvem pública apenas quando necessário para satisfazer as exigências.

### <a name="availability"></a>Disponibilidade

A implantação global tem os seus próprios desafios, como a conectividade variável e os diferentes regulamentos governamentais por região. Os desenvolvedores podem desenvolver apenas uma aplicação e depois implantá-la em diferentes razões com diferentes requisitos. Implemente a sua aplicação na nuvem pública Azure e, em seguida, implemente instâncias ou componentes adicionais localmente. Você pode gerir o tráfego entre todas as instâncias usando Azure.

### <a name="manageability"></a>Capacidade de gestão

#### <a name="a-single-consistent-development-approach"></a>Uma abordagem de desenvolvimento única e consistente

Azure e Azure Stack Hub permitem-lhe usar um conjunto consistente de ferramentas de desenvolvimento em toda a organização. Esta consistência facilita a implementação de uma prática de integração contínua e desenvolvimento contínuo (CI/CD). Muitas aplicações e serviços implantados no Azure ou no Azure Stack Hub são permutáveis e podem ser executados em qualquer um dos locais sem problemas.

Um gasoduto híbrido CI/CD pode ajudá-lo:

- Inicie uma nova construção baseada no código compromete-se com o seu repositório de código.
- Implemente automaticamente o seu código recém-construído para Azure para testes de aceitação do utilizador.
- Uma vez que o seu código tenha passado nos testes, desloque-se automaticamente para o Azure Stack Hub.

### <a name="a-single-consistent-identity-management-solution"></a>Uma única e consistente solução de gestão de identidade

O Azure Stack Hub trabalha com o Azure Ative Directory (Azure AD) e com os Serviços da Federação de Diretórios Ativos (ADFS). O Azure Stack Hub trabalha com a Azure AD em cenários conectados. Para ambientes que não têm conectividade, pode utilizar o ADFS como solução desconectada. Os principais do serviço são utilizados para garantir o acesso a apps, permitindo-lhes implementar ou configurar recursos através do Azure Resource Manager.

### <a name="security"></a>Segurança

#### <a name="ensure-compliance-and-data-sovereignty"></a>Garantir o cumprimento e a soberania dos dados

O Azure Stack Hub permite-lhe executar o mesmo serviço em vários países do que se usasse uma nuvem pública. A implementação da mesma aplicação em datacenters em cada país permite que os requisitos de soberania de dados sejam cumpridos. Esta capacidade garante que os dados pessoais são mantidos dentro das fronteiras de cada país.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub - postura de segurança

Não há nenhuma postura de segurança sem um processo de manutenção contínua e sólido. Por esta razão, a Microsoft investiu num motor de orquestração que aplica patches e atualiza perfeitamente em toda a infraestrutura.

Graças a parcerias com parceiros Azure Stack Hub OEM, a Microsoft alarga a mesma postura de segurança a componentes específicos do OEM, como o Hardware Lifecycle Host e o software que está a funcionar em cima dele. Esta parceria garante que o Azure Stack Hub tem uma postura de segurança uniforme e sólida em toda a infraestrutura. Por sua vez, os clientes podem construir e garantir as suas cargas de trabalho de aplicações.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Utilização de principais serviços através do portal PowerShell, CLI e Azure

Para dar acesso a um script ou app, crie uma identidade para a sua aplicação e autente a aplicação com as suas próprias credenciais. Esta identidade é conhecida como um diretor de serviço e permite-lhe:

- Atribua permissões à identidade da app que são diferentes das suas próprias permissões e estão restritas precisamente às necessidades da app.
- Utilize um certificado para autenticação ao executar um script automático.

Para obter mais informações sobre a criação principal do serviço e utilizar um certificado de credenciais, consulte [utilizar uma identidade de aplicação para aceder aos recursos.](/azure-stack/operator/azure-stack-create-service-principals)

## <a name="when-to-use-this-pattern"></a>Quando utilizar este padrão

- A minha organização está a usar uma abordagem de DevOps, ou tem uma planeada para o futuro próximo.
- Quero implementar práticas de CI/CD através da minha implementação do Azure Stack Hub e da nuvem pública.
- Quero consolidar o gasoduto CI/CD através de ambientes de nuvem e no local.
- Quero a capacidade de desenvolver aplicativos sem problemas usando serviços de nuvem ou no local.
- Quero aproveitar as habilidades consistentes do desenvolvedor através de aplicativos de nuvem e no local.
- Estou a usar o Azure, mas tenho programadores que estão a trabalhar numa nuvem Azure Stack Hub.
- As minhas aplicações no local experimentam picos de procura durante flutuações sazonais, cíclicas ou imprevisíveis.
- Tenho componentes no local e quero usar a nuvem para escaloná-los perfeitamente.
- Quero a escalabilidade das nuvens, mas quero que a minha aplicação seja executada o máximo possível no local.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Observe [aplicações de escala dinâmica entre datacenters e nuvem pública](https://www.youtube.com/watch?v=2lw8zOpJTn0) para uma visão geral de como este padrão é usado.
- Consulte [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para responder a perguntas adicionais que possa ter.
- Este padrão usa a família de produtos Azure Stack, incluindo o Azure Stack Hub. Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com o [guia de implementação da solução de escala cruzada (dados no local).](solution-deployment-guide-cross-cloud-scaling-onprem-data.md) O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.
