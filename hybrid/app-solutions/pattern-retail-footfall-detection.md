---
title: Padrão de deteção de rodapé usando Azure e Azure Stack Hub
description: Saiba como usar o Azure e o Azure Stack Hub para implementar uma solução de deteção de pés baseada em IA para analisar o tráfego de lojas de retalho.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895334"
---
# <a name="footfall-detection-pattern"></a>Padrão de deteção de rodapé

Este padrão fornece uma visão geral para a implementação de uma solução de deteção de pés baseada em IA para analisar o tráfego de visitantes em lojas de retalho. A solução gera insights de ações do mundo real, usando Azure Stack Hub e o Kit Dev Dev Visão Personalizada.

## <a name="context-and-problem"></a>Contexto e problema

A Contoso Stores gostaria de obter informações sobre como os clientes estão a receber os seus produtos atuais em relação ao layout da loja. Não conseguem colocar pessoal em todas as secções e é ineficiente que uma equipa de analistas reveja as filmagens de uma loja inteira. Além disso, nenhuma das suas lojas tem largura de banda suficiente para transmitir vídeos de todas as suas câmaras para a nuvem para análise.

A Contoso gostaria de encontrar uma forma discreta e amiga da privacidade de determinar a demografia, lealdade e reações dos seus clientes à loja de expositores e produtos.

## <a name="solution"></a>Solução

Este padrão de análise de retalho usa uma abordagem hierárquica para inferencing na borda. Ao utilizar o Kit Dev Dev Da Visão Personalizada, apenas imagens com rostos humanos são enviadas para análise para um Azure Stack Hub privado que gere os Serviços Cognitivos Azure. Dados anonimizados e agregados são enviados para a Azure para agregação em todas as lojas e visualização no Power BI. Combinar a borda e a nuvem pública permite que a Contoso aproveite a tecnologia moderna de IA, mantendo-se também em conformidade com as suas políticas corporativas e respeitando a privacidade dos seus clientes.

[![Solução de padrão de deteção de rodapé](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Aqui está um resumo de como a solução funciona:

1. O Kit Dev Dev Visão Personalizada obtém uma configuração do IoT Hub, que instala o IoT Edge Runtime e um modelo ML.
2. Se o modelo vir uma pessoa, tira uma fotografia e envia-a para o armazenamento de blob Azure Stack Hub.
3. O serviço blob aciona uma função Azure no Azure Stack Hub.
4. A Função Azure chama um recipiente com a API facial para obter dados demográficos e de emoção da imagem.
5. Os dados são anonimizados e enviados para um cluster Azure Event Hubs.
6. O cluster Event Hubs empurra os dados para stream Analytics.
7. Stream Analytics agrega os dados e empurra-os para o Power BI.

## <a name="components"></a>Componentes

Esta solução utiliza os seguintes componentes:

| Camada | Componente | Descrição |
|----------|-----------|-------------|
| Hardware na loja | [Kit de visão AI Dev personalizado](https://azure.github.io/Vision-AI-DevKit-Pages/) | Fornece filtragem na loja usando um modelo ML local que apenas captura imagens de pessoas para análise. A provisionada e atualizada de forma segura através do IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | O Azure Event Hubs fornece uma plataforma escalável para ingerir dados anonimizados que se integram perfeitamente com o Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Um trabalho do Azure Stream Analytics agrega os dados anonimizados e agrupe-os em janelas de 15 segundos para visualização. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | O Power BI fornece uma interface de painel fácil de usar para visualizar a saída do Azure Stream Analytics. |
| Azure Stack Hub | [Serviço de Aplicações](/azure-stack/operator/azure-stack-app-service-overview) | O fornecedor de recursos do Serviço de Aplicações (RP) fornece uma base para componentes de borda, incluindo funcionalidades de hospedagem e gestão para aplicações web/APIs e Funções. |
| | Cluster de motor de serviço Azure Kubernetes [(AKS)](https://github.com/Azure/aks-engine) | O AKS RP com AKS-Engine cluster implantado no Azure Stack Hub fornece um motor escalável e resistente para executar o recipiente Face API. |
| | Serviços Cognitivos Azure [enfrentam recipientes de API](/azure/cognitive-services/face/face-how-to-install-containers)| O Azure Cognitive Services RP com recipientes face API fornece deteção demográfica, emoção e visitante único na rede privada de Contoso. |
| | Armazenamento de Blobs | As imagens captadas no Kit AI Dev são enviadas para o armazenamento de bolhas do Azure Stack Hub. |
| | Funções do Azure | Uma Função Azure em execução no Azure Stack Hub recebe a entrada do armazenamento de bolhas e gere as interações com a API face. Emite dados anonimizados para um cluster de Centros de Eventos localizado em Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esta solução:

### <a name="scalability"></a>Escalabilidade

Para permitir que esta solução seja dimensionada em várias câmaras e locais, terá de se certificar de que todos os componentes conseguem lidar com a carga aumentada. Pode ter de tomar ações como:

- Aumente o número de unidades de streaming Stream Analytics.
- Dimensione a implantação da API face.
- Aumente a produção do cluster Event Hubs.
- Para casos extremos, pode ser necessário migrar de Azure Functions para uma máquina virtual.

### <a name="availability"></a>Disponibilidade

Uma vez que esta solução é hierárquico, é importante pensar em como lidar com falhas de rede ou de energia. Dependendo das necessidades do negócio, pode querer implementar um mecanismo para cache de imagens localmente e, em seguida, encaminhar para o Azure Stack Hub quando a conectividade voltar. Se a localização for suficientemente grande, a implantação de uma Borda de Caixa de Dados com o recipiente API face para esse local pode ser uma opção melhor.

### <a name="manageability"></a>Capacidade de gestão

Esta solução pode abranger muitos dispositivos e locais, o que pode ficar instável. [Os serviços IoT da Azure](/azure/iot-fundamentals/) podem ser usados para automaticamente trazer novos locais e dispositivos on-line e mantê-los atualizados.

### <a name="security"></a>Segurança

Esta solução captura imagens do cliente, tornando a segurança uma consideração primordial. Certifique-se de que todas as contas de armazenamento estão seguras com as políticas de acesso adequadas e rode regularmente as teclas. Garantir que as contas de armazenamento e os Centros de Eventos têm políticas de retenção que cumprem os regulamentos de privacidade corporativos e governamentais. Certifique-se também de que nivela os níveis de acesso ao utilizador. O tiering garante que os utilizadores apenas têm acesso aos dados de que necessitam para o seu papel.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Consulte o [padrão de dados tiered,](https://aka.ms/tiereddatadeploy)que é alavancado pelo padrão de deteção de pés.
- Consulte o [Kit Dev Dev Da Visão Personalizada](https://azure.github.io/Vision-AI-DevKit-Pages/) para saber mais sobre a utilização da visão personalizada. 

Quando estiver pronto para testar o exemplo da solução, continue com o [guia de deteção de footfall](solution-deployment-guide-retail-footfall-detection.md). O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.
