---
title: Dados hierárquicos para padrão de análise usando Azure e Azure Stack Hub
description: Aprenda a usar o Azure e o Azure Stack Hub para implementar uma solução de dados hierárquica através da nuvem híbrida.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911248"
---
# <a name="tiered-data-for-analytics-pattern"></a>Dados hierárquicos para padrão de análise

Este padrão ilustra como usar o Azure Stack Hub e o Azure para encenar, analisar, processar, higienizar e armazenar dados em vários locais no local e em nuvem.

## <a name="context-and-problem"></a>Contexto e problema

Um dos problemas que as organizações empresariais enfrentam no panorama tecnológico moderno diz respeito à segurança do armazenamento, processamento e análise de dados. Considerações incluem:

- conteúdo de dados
- localização
- requisitos de segurança e privacidade
- permissões de acesso
- manutenção
- armazenamento de armazenamento

O Azure, em combinação com o Azure Stack Hub, aborda as preocupações dos dados e oferece soluções de baixo custo. Esta solução é melhor expressa através de uma empresa de fabrico ou logística distribuída.

A solução baseia-se no seguinte cenário:

- Uma grande organização de fabrico de vários ramos.
- É necessário armazenamento, processamento e distribuição de dados rápidos e seguros entre as localizações remotas globais e a sua sede central.
- Atividade de funcionários e máquinas, informações de instalações e dados de reporte de negócios que devem permanecer seguros. Os dados devem ser distribuídos adequadamente e cumprir a política regional de conformidade e os regulamentos do sector.

## <a name="solution"></a>Solução

A utilização tanto no local como nos ambientes de nuvem pública satisfaz as exigências das empresas multi-instalações. O Azure Stack Hub oferece uma solução rápida, segura e flexível para recolher, processar, armazenar e distribuir dados locais e remotos. Este padrão é especialmente útil quando a segurança, confidencialidade, política corporativa e requisitos regulamentares podem diferir entre locais e utilizadores.

![Padrão de dados tiered para arquitetura de solução de analítica](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Componentes

Este padrão utiliza os seguintes componentes:

| Camada | Componente | Description |
|----------|-----------|-------------|
| Azure | Armazenamento | Uma conta [de Armazenamento Azure](/azure/storage/) fornece um ponto final de consumo de dados estéril. O Armazenamento do Azure é a solução de armazenamento da cloud da Microsoft para cenários de armazenamento de dados modernos. O Azure Storage oferece uma loja de objetos massivamente escalável para objetos de dados e um serviço de sistema de ficheiros para a nuvem. Também fornece uma loja de mensagens para mensagens confiáveis e uma loja NoSQL. |
| Azure Stack Hub | Armazenamento | Uma conta [de armazenamento Azure Stack Hub](/azure-stack/user/azure-stack-storage-overview) é usada para vários serviços:<br><br>- **Armazenamento de bolhas** para armazenamento de dados brutos. O armazenamento de blob pode conter qualquer tipo de texto ou dados binários, tais como um documento, ficheiro de mídia ou instalador de aplicações. Cada bolha é organizada debaixo de um contentor. Os contentores fornecem uma forma útil de atribuir políticas de segurança a grupos de objetos. Uma conta de armazenamento pode conter qualquer número de contentores, e um recipiente pode conter qualquer número de bolhas, até ao limite de capacidade de 500 TB da conta de armazenamento.<br>- **Armazenamento de bolhas** para arquivo de dados. Existem benefícios para o armazenamento de baixo custo para arquivar dados frescos. Exemplos de dados legais incluem backups, conteúdo sonoro, dados científicos, conformidade e dados de arquivo. Em geral, todos os dados acedidos com pouca frequência são considerados armazenamento fresco. Dados de tiering com base em atributos como frequência de acesso e período de retenção. Os dados do cliente são pouco frequentemente acedidos, mas requerem latência e desempenho semelhantes aos dados quentes.<br>- **Armazenamento de fila** para armazenamento de dados processados. O armazenamento de filas fornece mensagens em nuvem entre componentes de aplicações. Na conceção de apps para escala, os componentes de aplicações são muitas vezes dissociados para que possam escalar de forma independente. O armazenamento de filas fornece mensagens assíncronos para comunicação entre componentes de aplicações, quer estejam a correr na nuvem, no ambiente de trabalho, num servidor no local ou num dispositivo móvel. O Armazenamento de filas também suporta a gestão das tarefas assíncronas e a criação de fluxos de trabalho do processo. |
| | Funções do Azure | O serviço [Azure Functions](/azure/azure-functions/) é fornecido pelo [Serviço de Aplicações Azure no](/azure-stack/operator/azure-stack-app-service-overview) fornecedor de recursos Azure Stack Hub. As Funções Azure permitem executar o seu código num ambiente simples e sem servidor em resposta a uma variedade de eventos. Azure Functions escala para satisfazer a procura sem ter que criar um VM ou publicar uma aplicação web, usando a linguagem de programação à sua escolha. As funções são utilizadas pela solução para:<br><br>- **Ingestão de dados**<br>- **Esterilização de dados.** As funções ativadas manualmente podem executar o processamento, limpeza e arquivamento de dados programados. Exemplos podem incluir esfoliações de listas de clientes noturnos e processamento mensal de relatórios.|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esta solução:

### <a name="scalability"></a>Escalabilidade

A gama de funções e soluções de armazenamento Azure para atender às exigências de volume de dados e processamento. Para obter informações e metas de escalabilidade Azure, consulte [a documentação de escalabilidade do armazenamento Azure](/azure/storage/common/storage-scalability-targets).

### <a name="availability"></a>Disponibilidade

O armazenamento é a principal consideração de disponibilidade para este padrão. A ligação através de ligações rápidas é necessária para o processamento e distribuição de grandes volumes de dados.

### <a name="manageability"></a>Capacidade de gestão

A gestão desta solução depende da autoria de ferramentas de utilização e envolvimento do controlo de origem.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Consulte a documentação [do Azure Storage](/azure/storage/) and [Azure Functions.](/azure/azure-functions/) Este padrão faz uso pesado das contas de Armazenamento Azure e funções Azure tanto no Azure como no Azure Stack Hub.
- Consulte [considerações de design de aplicações híbridas](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter respostas a perguntas adicionais.
- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com os [dados Tiered para guia de implementação de solução analítica](https://aka.ms/tiereddatadeploy). O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.
