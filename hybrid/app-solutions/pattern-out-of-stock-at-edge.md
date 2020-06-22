---
title: Fora da deteção de stock usando Azure e Azure Stack Edge
description: Saiba como utilizar os serviços Azure e Azure Stack Edge para implementar fora da deteção de stock.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911192"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Fora da deteção de stock no padrão de borda

Este padrão ilustra como determinar se as prateleiras estão esgotadas usando um dispositivo Azure Stack Edge ou Azure IoT Edge e câmaras de rede.

## <a name="context-and-problem"></a>Contexto e problema

As lojas de retalho físico perdem vendas porque quando os clientes procuram um item, não está presente na prateleira. No entanto, o artigo poderia estar na parte de trás da loja e não ter sido reabastecido. As lojas gostariam de utilizar o seu pessoal de forma mais eficiente e ser automaticamente notificadas quando os itens precisam de reabastecimento.

## <a name="solution"></a>Solução

O exemplo da solução utiliza um dispositivo de borda, como um Azure Stack Edge em cada loja, que processa eficientemente dados de câmaras na loja. Este design otimizado permite que as lojas enviem apenas eventos e imagens relevantes para a nuvem. O design poupa largura de banda, espaço de armazenamento e garante a privacidade do cliente. À medida que os quadros são lidos a partir de cada câmara, um modelo ML processa a imagem e devolve qualquer uma das áreas de stock. A imagem e fora das áreas de stock são exibidas numa aplicação web local. Estes dados podem ser enviados para um ambiente de Insight da Série Temporal para mostrar insights no Power BI.

![Fora de stock na arquitetura de solução de borda](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Eis como funciona a solução:

1. As imagens são captadas a partir de uma câmara de rede sobre HTTP ou RTSP.
2. A imagem é redimensionada e enviada para o controlador de inferência, que comunica com o modelo ML para determinar se há alguma saída de imagens de stock.
3. O modelo ML devolve qualquer uma das áreas de stock.
4. O controlador de inferenculação envia a imagem em bruto para uma bolha (se especificada), e envia os resultados do modelo para o Azure IoT Hub e um processador de caixa de delimitação no dispositivo.
5. O processador de caixa de delimitação adiciona caixas de delimitação à imagem e cache o caminho da imagem numa base de dados de memória.
6. A aplicação web consulta para imagens e mostra-as na ordem recebida.
7. As mensagens do IoT Hub são agregadas no Time Series Insights.
8. Power BI exibe um relatório interativo de itens fora de stock ao longo do tempo com os dados da Time Series Insights.


## <a name="components"></a>Componentes

Esta solução utiliza os seguintes componentes:

| Camada | Componente | Description |
|----------|-----------|-------------|
| Hardware no local | Câmera de rede | É necessária uma câmara de rede, com um feed HTTP ou RTSP para fornecer as imagens para inferência. |
| Azure | Azure IoT Hub | [O Azure IoT Hub](/azure/iot-hub/) trata do fornecimento e mensagens dos dispositivos de borda. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) armazena as mensagens do IoT Hub para visualização. |
|  | Power BI | [O Microsoft Power BI](https://powerbi.microsoft.com/) fornece relatórios focados no negócio de eventos fora de stock. O Power BI fornece uma interface de painel fácil de usar para visualizar a saída do Azure Stream Analytics. |
| Azure Stack Edge ou<br>Dispositivo Azure IoT Edge | Azure IoT Edge | [O Azure IoT Edge](/azure/iot-edge/) orquestra o tempo de funcionamento dos contentores no local e trata da gestão e atualizações dos dispositivos.|
| | Onda cerebral do projeto Azure | Num dispositivo Azure Stack Edge, [o Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) utiliza arrays de porta programáveis de campo (FPGAs) para acelerar a inferição de ML.|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esta solução:

### <a name="scalability"></a>Escalabilidade

A maioria dos modelos de machine learning só pode funcionar com um certo número de fotogramas por segundo, dependendo do hardware fornecido. Determine a taxa de amostra ideal da(s) das suas câmaras para garantir que o gasoduto ML não recua. Diferentes tipos de hardware lidarão com diferentes números de câmaras e taxas de fotogramas.

### <a name="availability"></a>Disponibilidade

É importante considerar o que pode acontecer se o dispositivo de borda perder conectividade. Considere quais os dados que podem ser perdidos do painel Time Series Insights e Power BI. A solução de exemplo fornecida não é projetada para ser altamente disponível.

### <a name="manageability"></a>Capacidade de gestão

Esta solução pode abranger muitos dispositivos e locais, o que pode ficar instável. Os serviços IoT da Azure podem automaticamente trazer novos locais e dispositivos on-line e mantê-los atualizados. Devem também ser seguidos os procedimentos adequados de governação dos dados.

### <a name="security"></a>Segurança

Este padrão lida com dados potencialmente sensíveis. Certifique-se de que as chaves são regularmente rodadas e que as permissões na Conta de Armazenamento Azure e nas ações locais estão corretamente definidas.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:
- Vários serviços relacionados com IoT são utilizados neste padrão, incluindo [Azure IoT Edge,](/azure/iot-edge/) [Azure IoT Hub,](/azure/iot-hub/)e [Azure Time Series Insights](/azure/time-series-insights/).
- Para saber mais sobre o Microsoft Project Brainwave, consulte [o anúncio do blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) e confira o [Azure Accelerated Machine Learning com o vídeo do Project Brainwave](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Consulte [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter respostas a quaisquer perguntas adicionais.
- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com os [dados Tiered para guia de implementação de solução analítica](https://aka.ms/edgeinferencingdeploy). O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.
