---
title: Padrão de escala de nuvem cruzada no Azure Stack Hub
description: Aprenda a construir uma aplicação de nuvem cruzada escalável no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911136"
---
# <a name="cross-cloud-scaling-pattern"></a>Padrão de escala de nuvem cruzada

Adicione automaticamente recursos a uma aplicação existente para acomodar um aumento de carga.

## <a name="context-and-problem"></a>Contexto e problema

A sua aplicação não consegue aumentar a capacidade de atender a aumentos inesperados da procura. Esta falta de escalabilidade resulta em utilizadores que não chegam à aplicação durante os tempos de utilização máximo. A aplicação pode servir um número fixo de utilizadores.

As empresas globais requerem aplicações seguras, fiáveis e disponíveis baseadas na nuvem. Responder ao aumento da procura e utilizar as infraestruturas adequadas para apoiar essa procura é fundamental. As empresas lutam para equilibrar custos e manutenção com segurança de dados empresariais, armazenamento e disponibilidade em tempo real.

Pode não ser capaz de executar a sua aplicação na nuvem pública. No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária no seu ambiente no local para lidar com picos na procura da app. Com este padrão, pode utilizar a elasticidade da nuvem pública com a sua solução no local.

## <a name="solution"></a>Solução

O padrão de escala de nuvem cruzada estende uma aplicação localizada numa nuvem local com recursos de nuvem pública. O padrão é desencadeado por um aumento ou diminuição da procura, e respectivamente adiciona ou remove recursos na nuvem. Estes recursos proporcionam redundância, disponibilidade rápida e encaminhamento geocompatíveis.

![Padrão de escala de nuvem cruzada](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Este padrão aplica-se apenas aos componentes apátridas da sua aplicação.

## <a name="components"></a>Componentes

O padrão de escala de nuvem cruzada consiste nos seguintes componentes.

### <a name="outside-the-cloud"></a>Fora da nuvem

#### <a name="traffic-manager"></a>Gestor de Tráfego

No diagrama, este situa-se fora do grupo público de nuvens, mas teria de ser capaz de coordenar o tráfego tanto no centro de dados local como na nuvem pública. O equilibrador oferece uma elevada disponibilidade para aplicação, monitorizando pontos finais e fornecendo redistribuição de falha quando necessário.

#### <a name="domain-name-system-dns"></a>Sistema de Nomes de Domínio (DNS)

O Sistema de Nome de Domínio, ou DNS, é responsável por traduzir (ou resolver) um nome de website ou serviço para o seu endereço IP.

### <a name="cloud"></a>Cloud

#### <a name="hosted-build-server"></a>Servidor de construção hospedado

Um ambiente para hospedar o seu oleoduto de construção.

#### <a name="app-resources"></a>Recursos de aplicações

Os recursos da aplicação precisam de ser capazes de escalar e escalar, como conjuntos de escala de máquinas virtuais e contentores.

#### <a name="custom-domain-name"></a>Nome de domínio personalizado

Utilize um nome de domínio personalizado para pedidos de encaminhamento glob.

#### <a name="public-ip-addresses"></a>Endereços IP públicos

Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.  

### <a name="local-cloud"></a>Nuvem local

#### <a name="hosted-build-server"></a>Servidor de construção hospedado

Um ambiente para hospedar o seu oleoduto de construção.

#### <a name="app-resources"></a>Recursos de aplicações

Os recursos da aplicação precisam da capacidade de escalar e escalar, como conjuntos de escala de máquinas virtuais e contentores.

#### <a name="custom-domain-name"></a>Nome de domínio personalizado

Utilize um nome de domínio personalizado para pedidos de encaminhamento glob.

#### <a name="public-ip-addresses"></a>Endereços IP públicos

Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.

## <a name="issues-and-considerations"></a>Problemas e considerações

Na altura de decidir como implementar este padrão, considere os seguintes pontos:

### <a name="scalability"></a>Escalabilidade

O componente-chave da escala de nuvens cruzadas é a capacidade de realizar o dimensionamento a pedido. A escala deve ocorrer entre infraestruturas de nuvem pública e local e prestar um serviço consistente e fiável de acordo com a procura.

### <a name="availability"></a>Disponibilidade

Certifique-se de que as aplicações implementadas localmente são configuradas para alta disponibilidade através da configuração de hardware no local e implementação de software.

### <a name="manageability"></a>Capacidade de gestão

O padrão de nuvem cruzada garante uma gestão perfeita e uma interface familiar entre ambientes.

## <a name="when-to-use-this-pattern"></a>Quando utilizar este padrão

Utilize este padrão:

- Quando precisa de aumentar a sua capacidade de aplicação com exigências inesperadas ou exigências periódicas na procura.
- Quando não quer investir em recursos que só serão usados durante picos. Pague pelo que usa.

Este padrão não é recomendado quando:

- A sua solução requer que os utilizadores se conectem através da internet.
- O seu negócio tem regulamentos locais que exigem que a ligação originária venha de uma chamada no local.
- A sua rede experimenta estrangulamentos regulares que restringiriam o desempenho da escala.
- O seu ambiente está desligado da internet e não consegue alcançar a nuvem pública.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Consulte a visão geral do [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como funciona este equilibrador de carga de tráfego baseado em DNS.
- Consulte [considerações de design de aplicações híbridas](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter respostas para quaisquer perguntas adicionais.
- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com o [guia de implementação da solução de escala cross-cloud](solution-deployment-guide-cross-cloud-scaling.md). O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes. Aprende-se a criar uma solução cross-cloud para fornecer um processo manualmente desencadeado para mudar de uma aplicação web hospedada do Azure Stack Hub para uma aplicação web hospedada aZure. Também aprende a utilizar autoscaling através do gestor de tráfego, garantindo uma utilidade de nuvem flexível e escalável quando está carregada.
