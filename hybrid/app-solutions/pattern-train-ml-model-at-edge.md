---
title: Modelo de aprendizagem de máquina de trem no padrão de borda
description: Aprenda a fazer o treino de modelo de machine learning no limite com Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911097"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>Modelo de aprendizagem de máquina de trem no padrão de borda

Gerem modelos portáteis de aprendizagem automática (ML) a partir de dados que só existem no local.

## <a name="context-and-problem"></a>Contexto e problema

Muitas organizações gostariam de desbloquear insights a partir dos seus dados no local ou do legado usando ferramentas que os seus cientistas de dados entendem. [A Azure Machine Learning](/azure/machine-learning/) fornece ferramentas nativas em nuvem para treinar, sintonizar e implementar ml e modelos de aprendizagem profunda.  

No entanto, alguns dados são muito grandes enviados para a nuvem ou não podem ser enviados para a nuvem por razões regulamentares. Usando este padrão, os cientistas de dados podem usar a Azure Machine Learning para treinar modelos usando dados no local e computar.

## <a name="solution"></a>Solução

O treinamento no padrão de borda usa uma máquina virtual (VM) em execução no Azure Stack Hub. O VM está registado como um alvo de computação em Azure ML, permitindo-lhe aceder apenas aos dados disponíveis no local. Neste caso, os dados são armazenados no armazenamento de bolhas do Azure Stack Hub.

Uma vez treinado o modelo, é registado com Azure ML, contentorizado, e adicionado a um Registo de Contentores Azure para implantação. Para esta iteração do padrão, o VM de treino Azure Stack Hub deve ser acessível através da internet pública.

[![Modelo de ML de trem na arquitetura de borda](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

Eis como funciona o padrão:

1. O Azure Stack Hub VM é implantado e registado como alvo de computação com a Azure ML.
2. Uma experiência é criada em Azure ML que usa o Azure Stack Hub VM como alvo de computação.
3. Uma vez treinado o modelo, é registado e contentorizado.
4. O modelo pode agora ser implantado em locais que estejam no local ou na nuvem.

## <a name="components"></a>Componentes

Esta solução utiliza os seguintes componentes:

| Camada | Componente | Description |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [A Azure Machine Learning](/azure/machine-learning/) orquestra a formação do modelo ML. |
| | Registo de Contentores do Azure | A Azure ML embala o modelo num recipiente e armazena-o num [registo de contentores Azure](/azure/container-registry/) para implantação.|
| Azure Stack Hub | Serviço de Aplicações | [O Azure Stack Hub com o App Service](/azure-stack/operator/azure-stack-app-service-overview) fornece a base para os componentes na borda. |
| | Computação | Um Azure Stack Hub VM que executa Ubuntu com Docker é usado para treinar o modelo ML. |
| | Armazenamento | Os dados privados podem ser hospedados no armazenamento de blob Azure Stack Hub. |

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esta solução:

### <a name="scalability"></a>Escalabilidade

Para permitir esta solução à escala, terá de criar um VM de tamanho adequado no Azure Stack Hub para treinar.

### <a name="availability"></a>Disponibilidade

Certifique-se de que os scripts de treino e o Azure Stack Hub VM têm acesso aos dados no local utilizados para o treino.

### <a name="manageability"></a>Capacidade de gestão

Certifique-se de que os modelos e experiências estão devidamente registados, versados e marcados para evitar confusões durante a implementação do modelo.

### <a name="security"></a>Segurança

Este padrão permite ao Azure ML aceder a possíveis dados sensíveis no local. Certifique-se de que a conta utilizada para SSH no Azure Stack Hub VM tem uma palavra-passe forte e os scripts de treino não preservam ou carregam dados para a nuvem.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Consulte a [documentação de Aprendizagem automática Azure](/azure/machine-learning) para uma visão geral do ML e tópicos relacionados.
- Consulte [o Registo de Contentores Azure](/azure/container-registry/) para aprender a construir, armazenar e gerir imagens para implantações de contentores.
- Consulte o [Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) para saber mais sobre o fornecedor de recursos e como implementar.
- Consulte [considerações de design de aplicações híbridas](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter quaisquer perguntas adicionais respondidas.
- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com o [modelo ML do comboio no guia de implantação de bordas](https://aka.ms/edgetrainingdeploy). O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.
