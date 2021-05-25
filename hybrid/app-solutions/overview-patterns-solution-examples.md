---
title: Padrões híbridos e exemplos de solução para Azure e Azure Stack Hub
description: Uma visão geral dos padrões híbridos e exemplos de solução para aprender e construir soluções híbridas no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343863"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Padrões e exemplos de soluções híbridas para Azure e Azure Stack

A Microsoft fornece produtos e soluções Azure Stack como um ecossistema Azure consistente. A família Microsoft Azure Stack é uma extensão do Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>A nuvem híbrida e aplicativos híbridos

O Azure Stack traz a agilidade da computação em nuvem para o seu ambiente no local e a borda, permitindo uma *nuvem híbrida.* Azure Stack Hub, Azure Stack HCI e Azure Stack Edge estendem o Azure da nuvem para os seus datacenters soberanos, sucursais, campo e além. Com este conjunto diversificado de capacidades, pode:

- Reutilizar código e executar aplicações nativas de nuvem consistentemente em Azure e seus ambientes no local.
- Executar cargas de trabalho virtualizadas tradicionais com ligações opcionais aos serviços Azure.
- Transfira dados para a nuvem ou guarde-os no seu centro de dados soberano para manter a conformidade.
- Executar cargas de trabalho de aprendizagem de máquinas aceleradas por hardware, contentorizadas ou virtualizadas, tudo na borda inteligente.

As aplicações que abrangem nuvens também são referidas como *aplicações híbridas.* Pode construir aplicativos híbridos em nuvem em Azure e implantá-los no seu datacenter conectado ou desligado localizado em qualquer lugar.

Os cenários de aplicações híbridas variam muito com os recursos disponíveis para o desenvolvimento. Abrangem também considerações como geografia, segurança, acesso à Internet, entre outros. Embora os padrões e exemplos de solução aqui descritos possam não abordar todos os requisitos, eles fornecem orientações e exemplos para explorar e reutilizar enquanto implementam soluções híbridas.

## <a name="solution-patterns"></a>Padrões de solução

Padrões de solução abatem orientação de design repetível generalizada, a partir de cenários e experiências de clientes do mundo real. Um padrão é abstrato, permitindo que seja aplicável a diferentes tipos de cenários ou indústrias verticais. Cada padrão documenta o contexto e o problema, e fornece uma visão geral de um exemplo de solução. O exemplo da solução pretende ser uma possível implementação do padrão.

Existem dois tipos de artigos de padrão:

- Padrão único: fornece orientação de design para um único cenário de finalidade geral.
- Multi-padrão: fornece orientação de design onde a aplicação de múltiplos padrões é usada. Este padrão é frequentemente necessário para resolver cenários mais complexos ou problemas específicos da indústria.

## <a name="solution-deployment-guides"></a>Guias de implementação de soluções

Guias de implantação passo a passo ajudam na implementação de um exemplo de solução. O guia também pode consultar uma amostra de código de acompanhante, armazenada na amostra de amostras de [soluções](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)GitHub .

## <a name="next-steps"></a>Passos seguintes

- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.
- Explore as secções "Padrões" e "Guias de implementação de soluções" do TOC para saber mais sobre cada um.
- Leia sobre considerações de design de [aplicativos Híbridos](overview-app-design-considerations.md) para rever pilares de qualidade de software para projetar, implementar e operar aplicações híbridas.
- [Crie um ambiente de desenvolvimento no Azure Stack](/azure-stack/user/azure-stack-dev-start) e [implemente a sua primeira aplicação](/azure-stack/user/azure-stack-dev-start-deploy-app) no Azure Stack.
