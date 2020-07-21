---
title: Implemente uma app que escala a nuvem cruzada em Azure e Azure Stack Hub
description: Saiba como implementar uma aplicação que escala a nuvem cruzada no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 10cb042e2c6d0c6cb567e14072cd80bc663d686c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477342"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Implemente uma aplicação que escala a nuvem cruzada usando O Azure e Azure Stack Hub

Aprenda a criar uma solução cross-cloud para fornecer um processo manualmente desencadeado para mudar de uma aplicação web hospedada Azure Stack para uma aplicação web hospedada a Azure com autoscaling através do gestor de tráfego. Este processo garante uma utilidade de nuvem flexível e escalável quando está carregada.

Com este padrão, o seu inquilino pode não estar pronto para executar a sua app na nuvem pública. No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária no seu ambiente no local para lidar com picos na procura da app. O seu inquilino pode fazer uso da elasticidade da nuvem pública com a sua solução no local.

Nesta solução, você construirá um ambiente de amostra para:

> [!div class="checklist"]
> - Crie uma aplicação web multi-nó.
> - Configure e gere o processo de Implementação Contínua (CD).
> - Publique a aplicação web no Azure Stack Hub.
> - Criar uma libertação.
> - Aprenda a monitorizar e a rastrear as suas implementações.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

- Subscrição do Azure. Se necessário, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
- Um sistema integrado Azure Stack Hub ou implementação do Azure Stack Development Kit (ASDK).
  - Para obter instruções sobre a instalação do Azure Stack Hub, consulte [instalar o ASDK](/azure-stack/asdk/asdk-install.md).
  - Para um script de automatização pós-implantação ASDK, vá a:[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Esta instalação pode requerer algumas horas para ser concluída.
- Implementar serviços paaS do Serviço de [Aplicações](/azure-stack/operator/azure-stack-app-service-deploy.md) para o Azure Stack Hub.
- [Crie planos/ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) dentro do ambiente Azure Stack Hub.
- [Crie subscrição de inquilino](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) dentro do ambiente Azure Stack Hub.
- Crie uma aplicação web dentro da subscrição do inquilino. Tome nota do novo URL da aplicação web para utilização posterior.
- Implementar a máquina virtual Azure Pipelines (VM) dentro da subscrição do inquilino.
- É necessário windows server 2016 VM com .NET 3.5. Este VM será construído na subscrição do inquilino no Azure Stack Hub como o agente de construção privada.
- [O Windows Server 2016 com imagem VM SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) está disponível no Azure Stack Hub Marketplace. Se esta imagem não estiver disponível, trabalhe com um Azure Stack Hub Operator para garantir que é adicionada ao ambiente.

## <a name="issues-and-considerations"></a>Problemas e considerações

### <a name="scalability"></a>Escalabilidade

O componente-chave do escalonamento de nuvens cruzadas é a capacidade de fornecer escalas imediatas e a pedido entre infraestruturas de nuvem públicas e no local, fornecendo um serviço consistente e fiável.

### <a name="availability"></a>Disponibilidade

Certifique-se de que as aplicações implementadas localmente são configuradas para alta disponibilidade através da configuração de hardware no local e implementação de software.

### <a name="manageability"></a>Capacidade de gestão

A solução cross-cloud garante uma gestão perfeita e uma interface familiar entre ambientes. O PowerShell é recomendado para a gestão de plataformas cruzadas.

## <a name="cross-cloud-scaling"></a>Dimensionamento entre clouds

### <a name="get-a-custom-domain-and-configure-dns"></a>Obtenha um domínio personalizado e configuure DNS

Atualize o ficheiro da zona DNS para o domínio. A Azure AD verificará a propriedade do nome de domínio personalizado. Utilize [o Azure DNS](/azure/dns/dns-getstarted-portal) para registos DNS Azure/Office 365/external DNS dentro do Azure, ou adicione a entrada de DNS [num registo DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registe um domínio personalizado com um registo público.
2. Inicie sessão na entidade de registo de nome de domínio para o domínio. Um administrador aprovado pode ser necessário para fazer atualizações dns.
3. Atualize o ficheiro da zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD. (A entrada de DNS não afetará o encaminhamento de e-mail ou comportamentos de hospedagem web.)

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Crie uma aplicação web multi-nól padrão no Azure Stack Hub

Confule a integração contínua híbrida e a implementação contínua (CI/CD) para implementar aplicações web para Azure e Azure Stack Hub e para alterar automaticamente ambas as nuvens.

> [!Note]  
> Azure Stack Hub com imagens adequadas sincronizadas para executar (Windows Server e SQL) e implementação do Serviço de Aplicações são necessárias. Para mais informações, reveja a documentação do Serviço de Aplicações [Pré-requisitos para a implementação do Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

### <a name="add-code-to-azure-repos"></a>Adicionar código a Azure Repos

Repositórios do Azure

1. Inscreva-se no Azure Repos com uma conta que tem direitos de criação de projetos em Azure Repos.

    O CI/CD híbrido pode aplicar-se tanto ao código de aplicação como ao código de infraestrutura. Use [modelos de Gestor de Recursos Azure](https://azure.microsoft.com/resources/templates/) para o desenvolvimento de nuvem privada e hospedada.

    ![Ligue-se a um projeto em Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Clone o repositório** criando e abrindo a aplicação web padrão.

    ![Clone repo em app web Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Criar implementação de aplicativos web autossuficientes para Serviços de Aplicações em ambas as nuvens

1. Editar o ficheiro **WebApplication.csproj.** Selecione `Runtimeidentifier` e adicione `win10-x64` . (Ver [documentação de implantação independente.)](/dotnet/core/deploying/deploy-with-vs#simpleSelf)

    ![Editar o ficheiro do projeto de aplicativo web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Consulte o código para Azure Repos usando o Team Explorer.

3. Confirme que o código da aplicação foi verificado no Azure Repos.

## <a name="create-the-build-definition"></a>Criar a definição de construção

1. Inscreva-se nos Pipelines Azure para confirmar a capacidade de criar definições de construção.

2. Adicione **-r ganhar código 10-x64.** Esta adição é necessária para desencadear uma implantação autossuficiente com .NET Core.

    ![Adicione código à aplicação web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Executar a construção. O processo [de construção de implantação independente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefactos que funcionam no Azure e no Azure Stack Hub.

## <a name="use-an-azure-hosted-agent"></a>Use um agente azure hospedado

Usar um agente de construção hospedado em Azure Pipelines é uma opção conveniente para construir e implementar aplicações web. A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.

### <a name="manage-and-configure-the-cd-process"></a>Gerir e configurar o processo de CD

A Azure Pipelines e a Azure DevOps Services fornecem um oleoduto altamente configurável e manejável para libertações para vários ambientes como desenvolvimento, encenação, QA e ambientes de produção; incluindo a necessidade de aprovações em fases específicas.

## <a name="create-release-definition"></a>Criar definição de libertação

1. Selecione o botão **mais** para adicionar uma nova versão no separador **Versões** na secção **'Construir e Soltar'** serviços Azure DevOps.

    ![Criar uma definição de versão](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Aplique o modelo de implementação do serviço de aplicação da Azure.

   ![Aplicar o modelo de implementação do serviço de aplicação da Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. Under **Add artifact**, adicione o artefacto para a aplicação de construção Azure Cloud.

   ![Adicione artefacto à construção da Nuvem Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. No separador Pipeline, selecione a **Fase,** Ligação de tarefa do ambiente e desave os valores do ambiente em nuvem Azure.

   ![Definir valores do ambiente em nuvem Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Desaponte o **nome do ambiente** e selecione a **subscrição Azure** para o ponto final da Nuvem Azure.

      ![Selecione a subscrição do Azure para o ponto final da Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. No **nome do serviço app,** desaprote o nome de serviço de aplicação Azure necessário.

      ![Definir nome de serviço de aplicativo Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Insira "Hosted VS2017" na **fila do Agente** para o ambiente hospedado na nuvem Azure.

      ![Definir a fila do agente para o ambiente hospedado na nuvem Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. No menu Implementar o Serviço de Aplicações Azure, selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a **localização da pasta**.
  
      ![Selecione pacote ou pasta para ambiente de Serviço de Aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selecione pacote ou pasta para ambiente de Serviço de Aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Guarde todas as alterações e volte a **lançar o gasoduto**.

    ![Guardar alterações no gasoduto de libertação](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Adicione um novo artefacto selecionando a construção para a aplicação Azure Stack Hub.

    ![Adicione novo artefacto para a aplicação Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Adicione mais um ambiente aplicando a Implementação do Serviço de Aplicações Azure.

    ![Adicionar ambiente à implementação do Serviço de Aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Nomeie o novo ambiente "Azure Stack".

    ![Ambiente de nome na implementação do serviço de aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Encontre o ambiente Azure Stack no **separador Tarefa.**

    ![Ambiente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Selecione a subscrição para o ponto final Azure Stack.

    ![Selecione a subscrição para o ponto final Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Desaprote o nome da aplicação web Azure Stack como o nome do serviço de aplicação da App.
    ![Definir nome de aplicativo web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Selecione o agente Azure Stack.

    ![Selecione o agente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Na secção 'Implementar serviço de aplicações Azure', selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a localização da pasta.

    ![Selecione pasta para implementação do serviço de aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecione pasta para implementação do serviço de aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. No separador Variável adicione uma variável `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` nomeada, definir o seu valor como **verdadeiro**, e âmbito para Azure Stack.

    ![Adicionar variável à implementação da app Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Selecione o ícone de disparo de implementação **contínua** em ambos os artefactos e ative o gatilho de implementação **Continua.**

    ![Selecione o gatilho de implementação contínua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Selecione o ícone de condições **de pré-implantação** no ambiente Azure Stack e desaccione o gatilho para **depois de ser lançado.**

    ![Selecione condições de pré-implantação](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Guarde todas as alterações.

> [!Note]  
> Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis ambientais](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição de libertação a partir de um modelo. Estas definições não podem ser modificadas nas definições de tarefa; em vez disso, o item ambiente parental deve ser selecionado para editar estas definições.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Publicar no Azure Stack Hub via Visual Studio

Ao criar pontos finais, uma construção de Serviços Azure DevOps pode implementar aplicações do Serviço Azure para o Azure Stack Hub. A Azure Pipelines conecta-se ao agente de construção, que se conecta ao Azure Stack Hub.

1. Inscreva-se nos Serviços Azure DevOps e vá à página de definições da aplicação.

2. Nas **Definições**, selecione **Security**.

3. Nos **grupos VSTS,** selecione **Criadores de Pontos Finais**.

4. No separador **Membros,** **selecione Adicionar**.

5. In **Adicionar utilizadores e grupos,** introduza um nome de utilizador e selecione esse utilizador na lista de utilizadores.

6. Selecione **Guardar alterações**.

7. Na lista de **grupos VSTS,** selecione **Administradores de pontos finais**.

8. No separador **Membros,** **selecione Adicionar**.

9. In **Adicionar utilizadores e grupos,** introduza um nome de utilizador e selecione esse utilizador na lista de utilizadores.

10. Selecione **Guardar alterações**.

Agora que a informação do ponto final existe, a ligação Azure Pipelines para Azure Stack Hub está pronta a ser utilizada. O agente de construção em Azure Stack Hub recebe instruções da Azure Pipelines e, em seguida, o agente transmite informações de ponto final para comunicação com o Azure Stack Hub.

## <a name="develop-the-app-build"></a>Desenvolver a construção de aplicativos

> [!Note]  
> Azure Stack Hub com imagens adequadas sincronizadas para executar (Windows Server e SQL) e implementação do Serviço de Aplicações são necessárias. Para obter mais informações, consulte [Pré-requisitos para a implementação do Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

Utilize [modelos de Gestor de Recursos Azure](https://azure.microsoft.com/resources/templates/) como código de aplicação web do Azure Repos para implementar em ambas as nuvens.

### <a name="add-code-to-an-azure-repos-project"></a>Adicione código a um projeto Azure Repos

1. Inscreva-se no Azure Repos com uma conta que tem direitos de criação de projeto no Azure Stack Hub.

2. **Clone o repositório** criando e abrindo a aplicação web padrão.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Criar implementação de aplicativos web autossuficientes para Serviços de Aplicações em ambas as nuvens

1. Editar o ficheiro **WebApplication.csproj:** Selecione `Runtimeidentifier` e adicione `win10-x64` . Para obter mais informações, consulte a documentação [de implantação autónoma.](/dotnet/core/deploying/deploy-with-vs#simpleSelf)

2. Utilize o Team Explorer para verificar o código em Azure Repos.

3. Confirme que o código da aplicação foi verificado no Azure Repos.

### <a name="create-the-build-definition"></a>Criar a definição de construção

1. Inscreva-se nos Pipelines Azure com uma conta que pode criar uma definição de construção.

2. Aceda à página **Build Web Application** para o projeto.

3. Em **Arguments**, add **-r win10-x64** código. Esta adição é necessária para desencadear uma implantação independente com .NET Core.

4. Executar a construção. O processo [de construção de implantação independente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefactos que podem ser executados no Azure e no Azure Stack Hub.

#### <a name="use-an-azure-hosted-build-agent"></a>Use um agente de construção hospedado em Azure

Usar um agente de construção hospedado em Azure Pipelines é uma opção conveniente para construir e implementar aplicações web. A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.

### <a name="configure-the-continuous-deployment-cd-process"></a>Configure o processo de implantação contínua (CD)

A Azure Pipelines e a Azure DevOps Services fornecem um oleoduto altamente configurável e manejável para lançamentos para vários ambientes como desenvolvimento, encenação, garantia de qualidade (QA) e produção. Este processo pode incluir a necessidade de aprovações em fases específicas do ciclo de vida da aplicação.

#### <a name="create-release-definition"></a>Criar definição de libertação

Criar uma definição de lançamento é o passo final no processo de construção de aplicações. Esta definição de libertação é usada para criar uma libertação e implantar uma construção.

1. Inscreva-se na Azure Pipelines e vá para **Construir e Lançar** para o projeto.

2. No separador **Versões,** selecione **[ + ]** e, em seguida, escolha Criar **definição de desbloqueio**.

3. No **Selecionar um Modelo**, escolha a **implementação do serviço de aplicações Azure**e, em seguida, selecione **Apply**.

4. No **Add artifact**, a partir da **Definição Origem (Definição build)**, selecione a aplicação de construção Azure Cloud.

5. No **separador Pipeline,** selecione a ligação **1 Fase**, **1 Para** **Ver as tarefas ambientais**.

6. No separador **Tarefas, insira** O Azure como **o nome Ambiente** e selecione o EP AzureCloud Traders-Web da lista de **subscrições Azure.**

7. Introduza o nome de **serviço de aplicações Azure**, que está `northwindtraders` na próxima captura do ecrã.

8. Para a fase agente, **selecione Hosted VS2017** da lista de **filas do Agente.**

9. No **Serviço de Aplicações Deploy Azure**, selecione o **Pacote ou pasta** válido para o ambiente.

10. No **Ficheiro ou Pasta Select**, selecione **OK** para **localização**.

11. Guarde todas as alterações e volte ao **Pipeline.**

12. No **separador Pipeline,** **selecione Adicionar artefacto,** e escolha o **NorthwindCloud Traders-Vessel** da lista **Origem (Definição de Construção).**

13. No **Selecionar um Modelo,** adicione outro ambiente. Escolha **a aplicação do serviço de aplicações Azure** e, em seguida, selecione **Apply**.

14. Insira `Azure Stack Hub` como o nome **ambiente**.

15. No separador **Tarefas,** encontre e selecione O Hub Azure Stack.

16. A partir da lista **de subscrições do Azure,** selecione **AzureStack Traders-Vessel EP** para o ponto final do Azure Stack Hub.

17. Insira o nome da aplicação web Azure Stack Hub como o nome do **serviço app.**

18. Sob **a seleção de agente,** escolha **AzureStack -b Douglas Fir** da lista de filas do **Agente.**

19. Para **implementar o Serviço de Aplicações Azure**, selecione o Pacote ou **pasta** válido para o ambiente. No **Ficheiro ou Na Pasta Select**, selecione **OK** para a **localização**da pasta .

20. No **separador Variável,** encontre a variável `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` nomeada. Desa estale o valor variável para **verdadeiro,** e desa esta medida para **o Azure Stack Hub**.

21. No **separador Pipeline,** selecione o ícone **de disparo de implementação contínua** para o artefacto NorthwindCloud Traders-Web e desembra o gatilho de **implementação contínua** para **Ativado**. Faça o mesmo com o artefacto **northwindCloud traders-vessel.**

22. Para o ambiente Azure Stack Hub, selecione o ícone **de condições de pré-implantação** para definir o gatilho para **depois do lançamento**.

23. Guarde todas as alterações.

> [!Note]  
> Algumas definições para tarefas de libertação são definidas automaticamente como [variáveis ambientais](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição de libertação a partir de um modelo. Estas definições não podem ser modificadas nas definições de tarefa, mas podem ser modificadas nos itens do ambiente dos pais.

## <a name="create-a-release"></a>Criar um lançamento

1. No **separador Pipeline,** abra a lista **de Desbloqueio** e selecione **Criar verte**.

2. Introduza uma descrição para a libertação, verifique se os artefactos corretos estão selecionados e, em seguida, selecione **Criar**. Após alguns momentos, aparece um banner indicando que o novo lançamento foi criado e o nome de lançamento é apresentado como um link. Selecione o link para ver a página do resumo do lançamento.

3. A página do resumo do lançamento mostra detalhes sobre o lançamento. Na seguinte captura de ecrã para "Release-2", a secção **Ambiente** mostra o **estado de Implementação** do Azure como "EM PROGRESSO", e o estado de Azure Stack Hub é "SUCCEEDED". Quando o estado de implantação do ambiente Azure muda para "SUCCEEDED", aparece um banner indicando que o desbloqueio está pronto para aprovação. Quando uma implantação está pendente ou falhou, é mostrado um ícone de informação azul **(i).** Passe por cima do ícone para ver um pop-up que contém o motivo do atraso ou da falha.

4. Outras vistas, como a lista de lançamentos, também exibem um ícone que indica que a aprovação está pendente. O pop-up para este ícone mostra o nome do ambiente e mais detalhes relacionados com a implementação. É fácil para um administrador ver o progresso geral dos lançamentos e ver que lançamentos estão à espera de aprovação.

## <a name="monitor-and-track-deployments"></a>Monitorização e implantação de faixas

1. Na página de resumo **Do Lançamento-2,** selecione **Registos**. Durante uma implementação, esta página mostra o registo ao vivo do agente. O painel esquerdo mostra o estado de cada operação na implantação para cada ambiente.

2. Selecione o ícone da pessoa na coluna **Ação** para uma aprovação pré-implantação ou pós-implantação para ver quem aprovou (ou rejeitou) a implementação e a mensagem que forneceram.

3. Após o acabamento da colocação, todo o ficheiro de registo é apresentado no painel direito. Selecione qualquer **passo** no painel esquerdo para ver o ficheiro de registo para um único passo, como **Inicialize Job**. A capacidade de ver troncos individuais facilita o rastreio e depuração de partes da implantação geral. **Guarde** o ficheiro de registo para um passo ou **Descarregue todos os registos como zip**.

4. Abra o **separador Resumo** para ver informações gerais sobre o lançamento. Esta visão mostra detalhes sobre a construção, os ambientes para os quais foi implantado, estado de implantação e outras informações sobre a libertação.

5. Selecione uma ligação ambiental **(Azure** ou **Azure Stack Hub**) para ver informações sobre as implementações existentes e pendentes para um ambiente específico. Use estas vistas como uma forma rápida de verificar se a mesma construção foi implantada em ambos os ambientes.

6. Abra a **aplicação de produção implantada** num browser. Por exemplo, para o website Azure App Services, abra o URL `https://[your-app-name\].azurewebsites.net` .

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>Integração do Azure e do Azure Stack Hub proporciona uma solução de nuvem transversal escalável

Um serviço multi-nuvem flexível e robusto proporciona segurança de dados, back up e redundância, disponibilidade consistente e rápida, armazenamento e distribuição escaláveis e encaminhamento geocompatível. Este processo desencadeado manualmente garante uma troca de carga fiável e eficiente entre aplicações web hospedadas e disponibilidade imediata de dados cruciais.

## <a name="next-steps"></a>Passos seguintes

- Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)
