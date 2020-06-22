---
title: O padrão DevOps em Azure Stack Hub
description: Saiba mais sobre o padrão DevOps para que possa garantir a consistência através das implementações em Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 306cc9604a8e919724f9f76b7e5122d534d2d1ae
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911122"
---
# <a name="devops-pattern"></a>Padrão de devOps

Código a partir de um único local e implementar para vários alvos em ambientes de desenvolvimento, teste e produção que podem estar no seu datacenter local, nuvens privadas ou na nuvem pública.

## <a name="context-and-problem"></a>Contexto e problema

A continuidade, segurança e fiabilidade da implementação da aplicação são essenciais para as organizações e são fundamentais para as equipas de desenvolvimento.

As aplicações geralmente requerem código refactorado para ser executado em cada ambiente alvo. Isto significa que uma aplicação não é completamente portátil. Deve ser atualizado, testado e validado à medida que se move em cada ambiente. Por exemplo, o código escrito num ambiente de desenvolvimento deve então ser reescrito para trabalhar num ambiente de ensaio e reescrito quando finalmente aterra num ambiente de produção. Além disso, este código está especificamente ligado ao hospedeiro. Isto aumenta o custo e a complexidade da manutenção da sua app. Cada versão da aplicação está ligada a cada ambiente. O aumento da complexidade e da duplicação aumentam o risco de segurança e qualidade de código. Além disso, o código não pode ser facilmente redistribuído quando remove os anfitriões falhados ou implanta hospedeiros adicionais para lidar com aumentos na procura.

## <a name="solution"></a>Solução

O DevOps Pattern permite-lhe construir, testar e implementar uma aplicação que funciona em várias nuvens. Este padrão une a prática de integração contínua e entrega contínua. Com a integração contínua, o código é construído e testado sempre que um membro da equipa compromete uma mudança para o controlo da versão. A entrega contínua automatiza cada passo de uma construção para um ambiente de produção. Juntos, estes processos criam um processo de libertação que suporta a implantação em diversos ambientes. Com este padrão, você pode redigir o seu código e, em seguida, implementar o mesmo código para um ambiente de premissa, diferentes nuvens privadas, e as nuvens públicas. As diferenças de ambiente requerem uma alteração para um ficheiro de configuração em vez de alterações ao código.

![Padrão de devOps](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Com um conjunto consistente de ferramentas de desenvolvimento em todos os locais, nuvem privada e ambientes de nuvem pública, você pode implementar uma prática de integração contínua e entrega contínua. As aplicações e serviços implementados usando o DevOps Pattern são permutáveis e podem ser executados em qualquer um destes locais, tirando partido das funcionalidades e capacidades da nuvem pública.

A utilização de um gasoduto de libertação de DevOps ajuda-o:

- Iniciar uma nova construção baseada no código compromete-se a um único repositório.
- Implemente automaticamente o seu código recém-construído na nuvem pública para testes de aceitação do utilizador.
- Desloque-se automaticamente para uma nuvem privada uma vez que o seu código tenha passado nos testes.

## <a name="issues-and-considerations"></a>Problemas e considerações

O Padrão de DevOps destina-se a garantir a consistência entre as implementações, independentemente do ambiente alvo. No entanto, as capacidades variam entre ambientes de nuvens e no local. Considere os seguintes pontos:

- As funções, pontos finais, serviços e outros recursos na sua implantação estão disponíveis nos locais de implantação do alvo?
- Os artefactos de configuração são armazenados em locais acessíveis através das nuvens?
- Os parâmetros de implantação funcionarão em todos os ambientes-alvo?
- As propriedades específicas dos recursos estão disponíveis em todas as nuvens-alvo?

Para obter mais informações, consulte [os modelos do Gestor de Recursos Azure para obter consistência na nuvem.](https://docs.microsoft.com/azure/azure-resource-manager/templates-cloud-consistency)

Além disso, considere os seguintes pontos ao decidir como implementar este padrão:

### <a name="scalability"></a>Escalabilidade

Os sistemas de automação de implantação são o ponto de controlo chave nos Padrões DevOps. As implementações podem variar. A seleção do tamanho correto do servidor depende do tamanho da carga de trabalho esperada. Os VM custam mais à escala do que os contentores. Contudo, para utilizar contentores para dimensionamento, o processo de compilação tem de ser executado com contentores.

### <a name="availability"></a>Disponibilidade

Disponibilidade no contexto do DevPattern significa ser capaz de recuperar qualquer informação do estado associada ao seu fluxo de trabalho, tais como resultados de teste, dependências de códigos ou outros artefactos. Para avaliar os requisitos de disponibilidade, considere duas métricas comuns:

- O Objetivo do Tempo de Recuperação (RTO) especifica quanto tempo pode ficar sem um sistema.

- O Objetivo do Ponto de Recuperação (RPO) indica quantos dados pode perder se uma perturbação no serviço afetar o sistema.

Na prática, RTO e RPO implicam redundância e apoio. Na nuvem global de Azure, a disponibilidade não é uma questão de recuperação de hardware - que faz parte do Azure - mas sim garantir que mantém o estado dos seus sistemas DevOps. No Azure Stack Hub, a recuperação de hardware pode ser uma consideração.

Outra grande consideração na conceção do sistema utilizado para a automatização de implantação é o controlo de acessos e a correta gestão dos direitos necessários para a implantação de serviços em ambientes em nuvem. Que direitos são necessários para criar, eliminar ou modificar implementações? Por exemplo, um conjunto de direitos é normalmente necessário para criar um grupo de recursos em Azure e outro para implementar serviços no grupo de recursos.

### <a name="manageability"></a>Capacidade de gestão

O design de qualquer sistema baseado no padrão DevOps deve considerar a automação, o registo e o alerta para cada serviço em todo o portfólio. Use serviços partilhados, uma equipa de aplicação, ou ambos, e acompanhe também as políticas de segurança e a governação.

Implementar ambientes de produção e ambientes de desenvolvimento/teste em grupos de recursos separados no Azure ou no Azure Stack Hub. Em seguida, você pode monitorizar os recursos de cada ambiente e aumentar os custos de faturação por grupo de recursos. Também pode eliminar recursos como conjunto, o que é útil para implementações de testes.

## <a name="when-to-use-this-pattern"></a>Quando utilizar este padrão

Use este padrão se:

- Pode desenvolver código num ambiente que responda às necessidades dos seus desenvolvedores e implementar para um ambiente específico da sua solução onde possa ser difícil desenvolver um novo código.
- Pode utilizar o código e as ferramentas que os seus desenvolvedores gostariam, desde que sejam capazes de seguir o processo de integração contínua e entrega contínua no Padrão de DevOps.

Este padrão não é recomendado:

- Se não é possível automatizar infraestruturas, atear recursos, configuração, identidade e tarefas de segurança.
- Se as equipas não tiverem acesso a recursos híbridos em nuvem para implementar uma abordagem de Integração Contínua/Desenvolvimento Contínuo (CI/CD).

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Consulte a documentação do [Azure DevOps](/azure/devops) para saber mais sobre Azure DevOps e ferramentas relacionadas, incluindo Azure Repos e Azure Pipelines.
- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com o [guia de implementação de solução híbrida CI/CD de DevOps](https://aka.ms/hybriddevopsdeploy). O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes. Aprende-se a implementar uma aplicação para o Azure e o Azure Stack Hub utilizando um pipeline híbrido de integração contínua/entrega contínua (CI/CD).
