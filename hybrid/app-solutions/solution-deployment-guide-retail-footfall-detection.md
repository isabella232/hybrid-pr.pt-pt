---
title: Implementar solução de deteção de rodapé baseada em IA no Azure e no Azure Stack Hub
description: Saiba como implementar uma solução de deteção de rodapé baseada em IA para analisar o tráfego de visitantes em lojas de retalho usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901495"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Implementar uma solução de deteção de base de IA utilizando o Azure e o Azure Stack Hub

Este artigo descreve como implementar uma solução baseada em IA que gere insights de ações do mundo real usando Azure Stack Hub e o Kit Dev Dev Visão Personalizada.

Nesta solução, aprende-se a:

> [!div class="checklist"]
> - Implementar pacotes de aplicações nativas em nuvem (CNAB) na borda. 
> - Implemente uma aplicação que abra por entre limites de nuvens.
> - Utilize o Kit de Dev Vision AI Personalizado para inferência na borda.

> [!Tip]  
> ![Diagrama de pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Antes de começar com este guia de implantação, certifique-se de que:

- Reveja o tópico do [padrão de deteção de Footfall.](pattern-retail-footfall-detection.md)
- Obtenha acesso do utilizador a um Kit de Desenvolvimento de Pilhas Azure (ASDK) ou a instância integrada do Azure Stack Hub, com:
  - O [Serviço de Aplicações Azure no fornecedor de recursos Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview.md) instalado. Precisa de acesso do operador à sua instância do Azure Stack Hub ou trabalhar com o seu administrador para instalar.
  - Uma subscrição de uma oferta que fornece o Serviço de Aplicações e quota de armazenamento. Precisa de acesso do operador para criar uma oferta.
- Obtenha acesso a uma subscrição do Azure.
  - Se não tiver uma assinatura Azure, inscreva-se para uma [conta de teste gratuita](https://azure.microsoft.com/free/) antes de começar.
- Crie dois diretores de serviço no seu diretório:
  - Um conjunto para uso com recursos Azure, com acesso no âmbito de subscrição Azure.
  - Um conjunto para uso com recursos do Azure Stack Hub, com acesso no âmbito de subscrição do Azure Stack Hub.
  - Para saber mais sobre a criação de diretores de serviços e autorizar o acesso, consulte [utilizar uma identidade de aplicação para aceder aos recursos.](/azure-stack/operator/azure-stack-create-service-principals.md) Se preferir utilizar o Azure CLI, consulte [criar um rente-chefe de serviço Azure com Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).
- Implementar serviços cognitivos Azure em Azure ou Azure Stack Hub.
  - Primeiro, [saiba mais sobre os Serviços Cognitivos.](https://azure.microsoft.com/services/cognitive-services/)
  - Em [seguida, visite implementar os Serviços Cognitivos Azure para o Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) para implantar serviços cognitivos no Azure Stack Hub. Primeiro tem de se inscrever para ter acesso à pré-visualização.
- Clone ou descarregue um Azure Custom Vision AI Dev Kit não configurado. Para mais detalhes, consulte o [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).
- Inscreva-se numa conta Power BI.
- Uma chave de subscrição de Azure Cognitive Services Face API e URL de ponto final. Pode obter os dois com o teste gratuito [dos Serviços Cognitivos.](https://azure.microsoft.com/try/cognitive-services/?api=face-api) Ou, siga as instruções na [conta Criar um Serviço Cognitivo.](/azure/cognitive-services/cognitive-services-apis-create-account)
- Instalar os seguintes recursos de desenvolvimento:
  - [CLI 2.0 do Azure](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter, o](https://porter.sh/)que está a fazer? Você usa Porter para implementar aplicativos em nuvem usando manifestos de pacote CNAB que são fornecidos para si.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Azure IoT Tools para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Extensão python para código de estúdio visual](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>Implementar a aplicação de nuvem híbrida

Primeiro utiliza o Porter CLI para gerar um conjunto de credenciais e depois implementa a aplicação cloud.  

1. Clone ou descarregue o código de amostra de solução a partir de https://github.com/azure-samples/azure-intelligent-edge-patterns . 

1. Porter irá gerar um conjunto de credenciais que automatizarão a implementação da app. Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:

    - Um diretor de serviço para aceder aos recursos da Azure, incluindo o iD principal de serviço, chave, e DNS inquilino.
    - O ID de subscrição para a sua subscrição Azure.
    - Um diretor de serviço para aceder aos recursos do Azure Stack Hub, incluindo o iD principal de serviço, chave e DNS inquilino.
    - O ID de subscrição para a sua assinatura Azure Stack Hub.
    - Os seus serviços cognitivos Azure enfrentam a chave API e URL de ponto final de recurso.

1. Executar o processo de geração de credenciais Porter e seguir as instruções:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter também requer um conjunto de parâmetros para correr. Crie um ficheiro de texto de parâmetro e introduza os seguintes pares de nome/valor. Pergunte ao administrador do Azure Stack Hub se precisa de assistência com algum dos valores necessários.

   > [!NOTE] 
   > O `resource suffix` valor é usado para garantir que os recursos da sua implantação têm nomes únicos em todo o Azure. Deve ser uma cadeia única de letras e números, não mais do que 8 caracteres.

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   Guarde o ficheiro de texto e tome nota do seu percurso.

1. Está agora pronto para implementar a aplicação de nuvem híbrida usando o Porter. Executar o comando de instalação e assistir à medida que os recursos são implantados para O Azure e Azure Stack Hub:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. Uma vez concluída a implementação, tome nota dos seguintes valores:
    - A ligação da câmara.
    - A cadeia de ligação da conta de armazenamento de imagem.
    - Os nomes do grupo de recursos.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Preparar o Vision AI DevKit personalizado

Em seguida, configurar o Kit Dev Dev Vision AI personalizado, como mostrado no [quickstart Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). Também configura e testa a sua câmara, utilizando a cadeia de ligação fornecida no passo anterior.

## <a name="deploy-the-camera-app"></a>Implementar a aplicação da câmara

Utilize o Porter CLI para gerar um conjunto de credenciais e, em seguida, desloque a aplicação da câmara.

1. Porter irá gerar um conjunto de credenciais que automatizarão a implementação da app. Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:

    - Um diretor de serviço para aceder aos recursos da Azure, incluindo o iD principal de serviço, chave, e DNS inquilino.
    - O ID de subscrição para a sua subscrição Azure.
    - A cadeia de ligação da conta de armazenamento de imagem fornecida quando implementou a aplicação cloud.

1. Executar o processo de geração de credenciais Porter e seguir as instruções:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter também requer um conjunto de parâmetros para correr. Crie um ficheiro de texto de parâmetro e introduza o seguinte texto. Pergunte ao administrador do Azure Stack Hub se não conhece alguns dos valores necessários.

    > [!NOTE]
    > O `deployment suffix` valor é usado para garantir que os recursos da sua implantação têm nomes únicos em todo o Azure. Deve ser uma cadeia única de letras e números, não mais do que 8 caracteres.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Guarde o ficheiro de texto e tome nota do seu percurso.

4. Está agora pronto para implementar a aplicação da câmara usando o Porter. Executar o comando de instalação e observar à medida que a implementação IoT Edge é criada.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Verifique se a implementação da câmara está completa visualizando o feed da câmara `https://<camera-ip>:3000/` em , onde está o endereço IP da `<camara-ip>` câmara. Este passo pode demorar até 10 minutos.

## <a name="configure-azure-stream-analytics"></a>Configurar Azure Stream Analytics

Agora que os dados estão a fluir para o Azure Stream Analytics a partir da câmara, precisamos de autorizar manualmente que comunique com o Power BI.

1. A partir do portal Azure, abra **todos os recursos,** e o *processo-footfall seu trabalho de \[ \] sufixo.*

2. Na secção **Topologia da Tarefa** do painel de tarefas do Stream Analytics, selecione a opção **Saídas**.

3. Selecione o **lavatório de saída de saída de tráfego.**

4. Selecione **Renovar a autorização** e iniciar sôm no seu Power BI account.
  
    ![Renovar o pedido de autorização no Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Guarde as definições de saída.

6. Vá ao **painel de visão geral** e selecione **Comece** a enviar dados para Power BI.

7. Selecione **Agora** para a hora de início da saída da tarefa e selecione **Iniciar**. Pode ver o estado da tarefa na barra de notificação.

## <a name="create-a-power-bi-dashboard"></a>Criar um painel de instrumentos power BI

1. Assim que o trabalho tiver sucesso, vá ao [Power BI](https://powerbi.com/) e inscreva-se na sua conta de trabalho ou escola. Se a consulta de trabalho stream Analytics estiver a desausar resultados, o conjunto *de dados de base* que criou existe no **separador Datasets.**

2. A partir do seu espaço de trabalho Power BI, selecione **+ Criar** para criar um novo painel chamado *Footfall Analysis.*

3. Na parte superior da janela, selecione **Adicionar mosaico**. Em seguida, selecione **Dados de Transmissão em Fluxo Personalizados** e **Seguinte**. Escolha o **conjunto de dados de rodapé** nos seus **conjuntos de dados**. Selecione **o Cartão** a partir do **dropdown do tipo visualização** e adicione **a idade** aos **Campos**. Selecione **Seguinte** para introduzir um nome para o mosaico e, em seguida, selecione **Aplicar** para criar o mosaico.

4. Pode adicionar campos e cartões adicionais conforme desejado.

## <a name="test-your-solution"></a>Teste a sua solução

Observe como os dados nos cartões que criou no Power BI mudam à medida que pessoas diferentes andam em frente à câmara. As inferências podem demorar até 20 segundos a aparecer uma vez gravadas.

## <a name="remove-your-solution"></a>Remova a sua solução

Se quiser remover a sua solução, execute os seguintes comandos utilizando o Porter, utilizando os mesmos ficheiros de parâmetros que criou para a implementação:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Passos seguintes

- Saiba mais sobre [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md)
- Reveja e proponha melhorias [ao código para esta amostra no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
