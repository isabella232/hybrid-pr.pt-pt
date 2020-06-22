---
title: Padrão de aplicativo geo-distribuído no Azure Stack Hub
description: Saiba mais sobre o padrão de aplicação geo-distribuído para a borda inteligente usando Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910884"
---
# <a name="geo-distributed-app-pattern"></a>Padrão de aplicativo geo-distribuído

Saiba como fornecer pontos finais de aplicativos em várias regiões e encaminhar o tráfego de utilizadores com base nas necessidades de localização e conformidade.

## <a name="context-and-problem"></a>Contexto e problema

Organizações com geografias de grande alcance esforçam-se por distribuir de forma segura e precisa e permitir o acesso aos dados, garantindo ao mesmo tempo os níveis de segurança, conformidade e desempenho necessários por utilizador, localização e dispositivo além-fronteiras.

## <a name="solution"></a>Solução

O padrão de encaminhamento de tráfego geográfico Azure Stack Hub, ou aplicações geo-distribuídas, permite que o tráfego seja direcionado para pontos finais específicos com base em várias métricas. A criação de um Gestor de Tráfego com encaminhamento geográfico e configuração de pontos finais encaminha o tráfego para pontos finais com base nos requisitos regionais, regulação corporativa e internacional e necessidades de dados.

![Padrão geo-distribuído](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Componentes

### <a name="outside-the-cloud"></a>Fora da nuvem

#### <a name="traffic-manager"></a>Gestor de Tráfego

No diagrama, o Traffic Manager está localizado fora da nuvem pública, mas precisa de ser capaz de coordenar o tráfego tanto no centro de dados local como na nuvem pública. O equilibrador encaminha o tráfego para locais geográficos.

#### <a name="domain-name-system-dns"></a>Sistema de Nomes de Domínio (DNS)

O Sistema de Nome de Domínio, ou DNS, é responsável por traduzir (ou resolver) um nome de website ou serviço para o seu endereço IP.

### <a name="public-cloud"></a>Cloud pública

#### <a name="cloud-endpoint"></a>Ponto final da nuvem

Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.  

### <a name="local-clouds"></a>Nuvens locais

#### <a name="local-endpoint"></a>Ponto final local

Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.

## <a name="issues-and-considerations"></a>Problemas e considerações

Na altura de decidir como implementar este padrão, considere os seguintes pontos:

### <a name="scalability"></a>Escalabilidade

O padrão lida com o encaminhamento geográfico do tráfego em vez de escalar para fazer face aos aumentos de tráfego. No entanto, pode combinar este padrão com outras soluções Azure e no local. Por exemplo, este padrão pode ser usado com o Padrão de escala de nuvem cruzada.

### <a name="availability"></a>Disponibilidade

Certifique-se de que as aplicações implementadas localmente são configuradas para alta disponibilidade através da configuração de hardware no local e implementação de software.

### <a name="manageability"></a>Capacidade de gestão

O padrão garante uma gestão perfeita e uma interface familiar entre ambientes.

## <a name="when-to-use-this-pattern"></a>Quando utilizar este padrão

- A minha organização tem agências internacionais que exigem políticas de segurança e distribuição regionais personalizadas.
- Cada um dos escritórios da minha organização retira dados de empregados, negócios e instalações, exigindo atividade de reporte de acordo com os regulamentos locais e fuso horário.
- Requisitos de alta escala podem ser cumpridos escalando horizontalmente aplicações, com várias implementações de aplicações sendo feitas dentro de uma única região e em todas as regiões para lidar com requisitos de carga extrema.
- As aplicações devem estar altamente disponíveis e responder aos pedidos dos clientes mesmo em interrupções de uma região única.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Consulte a visão geral do [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como funciona este equilibrador de carga de tráfego baseado em DNS.
- Consulte [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter respostas para quaisquer perguntas adicionais.
- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com o [guia de implementação da solução de aplicações geo-distribuídas.](solution-deployment-guide-geo-distributed.md) O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes. Aprende-se a direcionar o tráfego para pontos finais específicos, com base em várias métricas utilizando o padrão de aplicação geo-distribuído. A criação de um perfil de Gestor de Tráfego com configuração geográfica de encaminhamento e ponto final garante que a informação é encaminhada para pontos finais com base nos requisitos regionais, na regulação corporativa e internacional e nas necessidades dos seus dados.
