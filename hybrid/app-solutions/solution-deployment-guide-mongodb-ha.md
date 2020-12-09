---
title: Implemente uma solução MongoDB altamente disponível para O Azure e Azure Stack Hub
description: Saiba como implementar uma solução MongoDB altamente disponível para o Azure e o Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 624f032def509d8e42d55807d72176e5fce85910
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901512"
---
# <a name="deploy-a-highly-available-mongodb-solution-across-two-azure-stack-hub-environments"></a>Implementar uma solução MongoDB altamente disponível em dois ambientes Azure Stack Hub

Este artigo irá passar por uma implementação automatizada de um cluster básico altamente disponível (HA) MongoDB com um site de recuperação de desastres (DR) em dois ambientes do Azure Stack Hub. Para saber mais sobre o MongoDB e a alta disponibilidade, consulte [os Membros conjuntos de réplicas.](https://docs.mongodb.com/manual/core/replica-set-members/)

Nesta solução, criará um ambiente de amostra para:

> [!div class="checklist"]
> - Orquestrou uma implantação através de dois Hubs Azure Stack.
> - Use o Docker para minimizar problemas de dependência com perfis AZure API.
> - Implemente um cluster mongoDB básico altamente disponível com um local de recuperação de desastres.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Arquitetura para MongoDB com Azure Stack Hub

![arquitetura mongoDB altamente disponível em Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Pré-requisitos para MongoDB com Azure Stack Hub

- Dois sistemas integrados Azure Stack Hub (Azure Stack Hub). Esta implementação não funciona no Kit de Desenvolvimento de Pilhas Azure (ASDK). Para saber mais sobre o Azure Stack Hub, veja [o que é O Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)
  - Uma subscrição de inquilino em cada Azure Stack Hub. 
  - **Tome nota de cada ID de subscrição e do ponto final do Azure Resource Manager para cada Azure Stack Hub.**
- Um diretor de serviço Azure Ative (Azure AD) que tem permissões para a subscrição do inquilino em cada Azure Stack Hub. Você pode precisar de criar dois principais de serviço se os Azure Stack Hubs forem implantados contra diferentes inquilinos AD Azure. Para aprender a criar um principal de serviço para o Azure Stack Hub, consulte [utilizar uma identidade de aplicação para aceder aos recursos do Azure Stack Hub.](/azure-stack/user/azure-stack-create-service-principals)
  - **Tome nota da identificação de cada pedido do diretor de serviço, segredo do cliente e nome do inquilino (xxxxx.onmicrosoft.com).**
- Ubuntu 16.04 sindicado para cada Azure Stack Hub's Marketplace. Para saber mais sobre a sindicalização do mercado, consulte [itens de Mercado de Descarregamento para Azure Stack Hub.](/azure-stack/operator/azure-stack-download-azure-marketplace-item)
- [Docker para Windows](https://docs.docker.com/docker-for-windows/) instalado na sua máquina local.

## <a name="get-the-docker-image"></a>Obtenha a imagem do Docker

As imagens do Docker para cada implementação eliminam problemas de dependência entre diferentes versões do Azure PowerShell.

1. Certifique-se de que o Docker para o Windows está a utilizar recipientes Windows.
2. Executar o seguinte comando num pedido de comando elevado para obter o recipiente Docker com os scripts de implantação.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Implementar os clusters

1. Uma vez puxada com sucesso a imagem do recipiente, inicie a imagem.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Uma vez iniciado o contentor, receberá um terminal PowerShell elevado no recipiente. Mude os diretórios para chegar ao script de implementação.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Executar a implantação. Forneça credenciais e nomes de recursos sempre que necessário. HA refere-se ao Azure Stack Hub onde o cluster HA será implantado. Dr refere-se ao Azure Stack Hub onde o cluster DR será implantado.

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. Escreva `Y` para permitir a instalação do fornecedor NuGet, que irá dar início aos módulos "2018-03-01-híbridos" a instalar.

5. Os recursos ha serão implantados primeiro. Monitorize a colocação e espere que termine. Assim que tiver a mensagem indicando que a implementação de HA está terminada, pode verificar o portal do HA Azure Stack Hub para ver os recursos utilizados.

6. Continue com a implementação de recursos DR e decida se gostaria de ativar uma caixa de salto no DR Azure Stack Hub para interagir com o cluster.

7. Aguarde que a implementação de recursos DR termine.

8. Uma vez terminada a implantação de recursos DR, saia do recipiente.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Passos seguintes

- Se ativou a caixa de salto VM no DR Azure Stack Hub, pode ligar-se via SSH e interagir com o cluster MongoDB instalando o mongo CLI. Para saber mais sobre a interação com o MongoDB, consulte [a Mongo Shell.](https://docs.mongodb.com/manual/mongo/)
- Para saber mais sobre aplicações híbridas em nuvem, consulte [a Hybrid Cloud Solutions.](/azure-stack/user/)
- Modifique o código para esta amostra no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
