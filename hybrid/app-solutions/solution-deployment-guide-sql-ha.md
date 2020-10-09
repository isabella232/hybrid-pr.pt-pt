---
title: Implementar um grupo de disponibilidade SQL Server 2016 para Azure e Azure Stack Hub
description: Saiba como implementar um grupo de disponibilidade SQL Server 2016 para O Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852478"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Implementar um grupo de disponibilidade SQL Server 2016 para Azure e Azure Stack Hub

Este artigo irá passar por uma implementação automatizada de um cluster básico altamente disponível (HA) SQL Server 2016 Enterprise com um site assíncrona de recuperação de desastres (DR) através de dois ambientes Azure Stack Hub. Para saber mais sobre o SQL Server 2016 e alta disponibilidade, consulte [sempre em grupos de disponibilidade: uma solução de alta disponibilidade e recuperação de desastres.](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)

Nesta solução, você construirá um ambiente de amostra para:

> [!div class="checklist"]
> - Orquestrou uma implantação através de dois Hubs Azure Stack.
> - Use o Docker para minimizar problemas de dependência com perfis AZure API.
> - Implemente um cluster empresarial SQL Server 2016 básico altamente disponível com um site de recuperação de desastres.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="architecture-for-sql-server-2016"></a>Arquitetura para SQL Server 2016

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Pré-requisitos para SQL Server 2016

- Dois sistemas integrados Azure Stack Hub (Azure Stack Hub). Esta implementação não funciona no Kit de Desenvolvimento de Pilhas Azure (ASDK). Para saber mais sobre o Azure Stack Hub, consulte a visão geral da [Azure Stack](https://azure.microsoft.com/overview/azure-stack/).
- Uma subscrição de inquilino em cada Azure Stack Hub.
  - **Tome nota de cada ID de subscrição e do ponto final do Azure Resource Manager para cada Azure Stack Hub.**
- Um diretor de serviço Azure Ative (Azure AD) que tem permissões para a subscrição do inquilino em cada Azure Stack Hub. Você pode precisar de criar dois principais de serviço se os Azure Stack Hubs forem implantados contra diferentes inquilinos AD Azure. Para aprender a criar um principal de serviço para o Azure Stack Hub, consulte [os principais dos serviços create para dar às aplicações acesso aos recursos do Azure Stack Hub.](/azure-stack/user/azure-stack-create-service-principals)
  - **Tome nota da identificação de cada pedido do diretor de serviço, segredo do cliente e nome do inquilino (xxxxx.onmicrosoft.com).**
- SQL Server 2016 Enterprise sindicalizado para cada Azure Stack Hub's Marketplace. Para saber mais sobre a sindicalização do mercado, consulte [itens de Mercado de Descarregamento para Azure Stack Hub.](/azure-stack/operator/azure-stack-download-azure-marketplace-item)
    **Certifique-se de que a sua organização tem as licenças SQL apropriadas.**
- [Docker para Windows](https://docs.docker.com/docker-for-windows/) instalado na sua máquina local.

## <a name="get-the-docker-image"></a>Obtenha a imagem do Docker

As imagens do Docker para cada implementação eliminam problemas de dependência entre diferentes versões do Azure PowerShell.

1. Certifique-se de que o Docker para o Windows está a utilizar recipientes Windows.
2. Execute o seguinte script num pedido de comando elevado para obter o recipiente Docker com os scripts de implantação.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Implementar o grupo de disponibilidade

1. Uma vez puxada com sucesso a imagem do recipiente, inicie a imagem.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Uma vez iniciado o contentor, receberá um terminal PowerShell elevado no recipiente. Mude os diretórios para chegar ao script de implementação.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Executar a implantação. Forneça credenciais e nomes de recursos sempre que necessário. HA refere-se ao Azure Stack Hub onde o cluster HA será implantado. Dr refere-se ao Azure Stack Hub onde o cluster DR será implantado.

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

5. Aguarde a conclusão da implementação dos recursos.

6. Uma vez concluída a implantação de recursos DR, saia do recipiente.

      ```powershell
      exit
      ```

7. Inspecione a implantação visualizando os recursos em cada portal do Azure Stack Hub. Ligue-se a uma das instâncias SQL sobre o ambiente HA e inspecione o Grupo de Disponibilidade através do SQL Server Management Studio (SSMS).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Passos seguintes

- Utilize o SQL Server Management Studio para falhar manualmente sobre o cluster. Consulte [executar uma falha manual forçada de um grupo de disponibilidade sempre disponível (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Saiba mais sobre aplicativos híbridos em nuvem. Consulte [soluções de nuvem híbrida.](/azure-stack/user/)
- Utilize os seus próprios dados ou modifique o código para esta amostra no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).