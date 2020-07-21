---
title: Tráfego direto com uma app geo-distribuída usando Azure e Azure Stack Hub
description: Saiba como direcionar o tráfego para pontos finais específicos com uma solução de aplicações geo-distribuídas utilizando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 741ddf2c3ed234788af359dd233f6a656fbea13c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477359"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Tráfego direto com uma app geo-distribuída usando Azure e Azure Stack Hub

Aprenda a direcionar o tráfego para pontos finais específicos com base em várias métricas usando o padrão de aplicações geo-distribuídas. A criação de um perfil de Gestor de Tráfego com configuração geográfica de encaminhamento e ponto final garante que a informação é encaminhada para pontos finais com base nos requisitos regionais, na regulação corporativa e internacional e nas necessidades dos seus dados.

Nesta solução, você construirá um ambiente de amostra para:

> [!div class="checklist"]
> - Crie uma aplicação geo-distribuída.
> - Use o Traffic Manager para direcionar a sua aplicação.

## <a name="use-the-geo-distributed-apps-pattern"></a>Use o padrão de aplicações geo-distribuídas

Com o padrão geo-distribuído, a sua aplicação abrange regiões. Pode falhar na nuvem pública, mas alguns dos seus utilizadores podem exigir que os seus dados permaneçam na sua região. Pode direcionar os utilizadores para a nuvem mais adequada com base nas suas necessidades.

### <a name="issues-and-considerations"></a>Problemas e considerações

#### <a name="scalability-considerations"></a>Considerações de escalabilidade

A solução que vai construir com este artigo não é para acomodar a escalabilidade. No entanto, se for utilizado em combinação com outras soluções Azure e no local, pode acomodar requisitos de escalabilidade. Para obter informações sobre a criação de uma solução híbrida com autoscaling através do gestor de tráfego, consulte [Criar soluções de escala cross-cloud com o Azure.](solution-deployment-guide-cross-cloud-scaling.md)

#### <a name="availability-considerations"></a>Considerações de disponibilidade

Como é o caso das considerações de escalabilidade, esta solução não aborda diretamente a disponibilidade. No entanto, a Azure e as soluções no local podem ser implementadas nesta solução para garantir uma elevada disponibilidade para todos os componentes envolvidos.

### <a name="when-to-use-this-pattern"></a>Quando utilizar este padrão

- A sua organização tem sucursais internacionais que exigem políticas de segurança e distribuição regionais personalizadas.

- Cada um dos escritórios da sua organização retira dados de funcionários, negócios e instalações, o que requer atividade de reporte de acordo com os regulamentos locais e fusos horários.

- Os requisitos de alta escala são cumpridos através da escala horizontal de aplicações com múltiplas implementações de aplicações dentro de uma única região e em todas as regiões para lidar com requisitos de carga extrema.

### <a name="planning-the-topology"></a>Planejando a topologia

Antes de construir uma pegada de aplicativo distribuída, ajuda a saber as seguintes coisas:

- **Domínio personalizado para a aplicação:** Qual é o nome de domínio personalizado que os clientes usarão para aceder à aplicação? Para a aplicação da amostra, o nome de domínio personalizado é *www \. scalableasedemo.com.*

- **Domínio do Gestor de Tráfego:** Um nome de domínio é escolhido ao criar um [perfil de Gestor de Tráfego Azure](/azure/traffic-manager/traffic-manager-manage-profiles). Este nome é combinado com o *sufixo* trafficmanager.net para registar uma entrada de domínio que é gerida pelo Traffic Manager. Para a aplicação da amostra, o nome escolhido é *escalável-ase-demo*. Como resultado, o nome de domínio completo que é gerido pelo Traffic Manager é *scalable-ase-demo.trafficmanager.net*.

- **Estratégia para escalar a pegada da aplicação:** Decida se a pegada da aplicação será distribuída por vários ambientes do Serviço de Aplicações numa única região, várias regiões ou uma mistura de ambas as abordagens. A decisão deve basear-se nas expectativas de onde o tráfego de clientes terá origem e em que ponto o resto da infraestrutura de back-end suportada de uma aplicação pode escalar. Por exemplo, com uma aplicação 100% apátrida, uma aplicação pode ser massivamente dimensionada usando uma combinação de múltiplos ambientes de Serviço de Aplicações por região de Azure, multiplicados por ambientes de Serviço de Aplicações implantados em várias regiões do Azure. Com mais de 15 regiões globais de Azure disponíveis para escolher, os clientes podem realmente construir uma pegada de aplicações de hiperescala em todo o mundo. Para a aplicação de amostras aqui utilizada, três ambientes de Serviço de Aplicações foram criados numa única região de Azure (South Central US).

- **Convenção de nomeação para os ambientes do Serviço de Aplicações:** Cada ambiente de Serviço de Aplicações requer um nome único. Além de um ou dois ambientes de Serviço de Aplicações, é útil ter uma convenção de nomeação para ajudar a identificar cada ambiente de Serviço de Aplicações. Para a aplicação de amostras usada aqui, foi usada uma simples convenção de nomeação. Os nomes dos três ambientes do Serviço de Aplicações são *fe1ase,* *fe2ase*e *fe3ase.*

- **Convenção de nomeação para as aplicações:** Uma vez que serão implementados vários casos da aplicação, é necessário um nome para cada instância da aplicação implementada. Com o App Service Environment para Aplicações de Energia, o mesmo nome de aplicação pode ser usado em vários ambientes. Uma vez que cada ambiente do Serviço de Aplicações tem um sufixo de domínio único, os desenvolvedores podem optar por reutilizar exatamente o mesmo nome de aplicação em cada ambiente. Por exemplo, um programador poderia ter apps nomeadas da seguinte forma: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, e assim por diante. Para a aplicação usada aqui, cada instância de aplicação tem um nome único. Os nomes de instâncias de aplicações utilizados são *webfrontend1,* *webfrontend2*e *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="part-1-create-a-geo-distributed-app"></a>Parte 1: Criar uma aplicação geo-distribuída

Nesta parte, vai criar uma aplicação web.

> [!div class="checklist"]
> - Crie aplicativos web e publique.
> - Adicione código ao Azure Repos.
> - Aponte a construção da aplicação para vários alvos em nuvem.
> - Gerir e configurar o processo de CD.

### <a name="prerequisites"></a>Pré-requisitos

É necessária uma subscrição Azure e uma instalação do Azure Stack Hub.

### <a name="geo-distributed-app-steps"></a>Passos de aplicação geo-distribuídos

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Obtenha um domínio personalizado e configuure DNS

Atualize o ficheiro da zona DNS para o domínio. A Azure AD pode então verificar a propriedade do nome de domínio personalizado. Utilize [o Azure DNS](/azure/dns/dns-getstarted-portal) para registos DNS Azure/Office 365/external DNS dentro do Azure, ou adicione a entrada de DNS [num registo DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registe um domínio personalizado com um registo público.

2. Inicie sessão na entidade de registo de nome de domínio para o domínio. Um administrador aprovado pode ser necessário para escruissar as atualizações do DNS.

3. Atualize o ficheiro da zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD. A entrada do DNS não altera comportamentos como o encaminhamento de correio ou o hospedagem na Web.

### <a name="create-web-apps-and-publish"></a>Criar aplicativos web e publicar

Confule a integração contínua híbrida/entrega contínua (CI/CD) para implementar a Web App para Azure e Azure Stack Hub, e empurre automaticamente as alterações em ambas as nuvens.

> [!Note]  
> Azure Stack Hub com imagens adequadas sincronizadas para executar (Windows Server e SQL) e implementação do Serviço de Aplicações são necessárias. Para obter mais informações, consulte [Pré-requisitos para a implementação do Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

#### <a name="add-code-to-azure-repos"></a>Adicionar código a Azure Repos

1. Inscreva-se no Visual Studio com uma **conta que tem direitos de criação de projetos** no Azure Repos.

    O CI/CD pode aplicar-se tanto ao código de aplicação como ao código de infraestrutura. Use [modelos de Gestor de Recursos Azure](https://azure.microsoft.com/resources/templates/) para o desenvolvimento de nuvem privada e hospedada.

    ![Conecte-se a um projeto em Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Clone o repositório** criando e abrindo a aplicação web padrão.

    ![Repositório de clones no Estúdio Visual](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Criar implementação de aplicativos web em ambas as nuvens

1. Editar o ficheiro **WebApplication.csproj:** Selecione `Runtimeidentifier` e adicione `win10-x64` . (Ver [documentação de implantação independente.)](/dotnet/core/deploying/deploy-with-vs#simpleSelf)

    ![Editar o ficheiro do projeto de aplicativo web no Estúdio Visual](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Consulte o código para Azure Repos** usando o Team Explorer.

3. Confirme que o código de **aplicação** foi verificado no Azure Repos.

### <a name="create-the-build-definition"></a>Criar a definição de construção

1. **Inscreva-se nos Pipelines Azure** para confirmar a capacidade de criar definições de construção.

2. Adicione `-r win10-x64` código. Esta adição é necessária para desencadear uma implantação autossuficiente com .NET Core.

    ![Adicione código à definição de construção em Gasodutos Azure](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Executar a construção.** O processo [de construção de implantação independente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefactos que podem ser executados no Azure e no Azure Stack Hub.

#### <a name="using-an-azure-hosted-agent"></a>Usando um agente azure hospedado

Usar um agente hospedado em Azure Pipelines é uma opção conveniente para construir e implementar aplicações web. A manutenção e as atualizações são executadas automaticamente pela Microsoft Azure, o que permite o desenvolvimento, teste e implementação ininterruptas.

### <a name="manage-and-configure-the-cd-process"></a>Gerir e configurar o processo de CD

Os Serviços Azure DevOps fornecem um oleoduto altamente configurável e manejável para libertações para vários ambientes, tais como desenvolvimento, encenação, QA e ambientes de produção; incluindo a necessidade de aprovações em fases específicas.

## <a name="create-release-definition"></a>Criar definição de libertação

1. Selecione o botão **mais** para adicionar uma nova versão no separador **Versões** na secção **'Construir e Soltar'** serviços Azure DevOps.

    ![Criar uma definição de lançamento nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image5.png)

2. Aplique o modelo de implementação do serviço de aplicação da Azure.

   ![Aplicar o modelo de implementação do serviço de aplicação da Azure em serviços de devops Azure](media/solution-deployment-guide-geo-distributed/image6.png)

3. Under **Add artifact**, adicione o artefacto para a aplicação de construção Azure Cloud.

   ![Adicione artefacto à Azure Cloud build em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. No separador Pipeline, selecione a **Fase,** Ligação de tarefa do ambiente e desave os valores do ambiente em nuvem Azure.

   ![Definir valores de ambiente em nuvem Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image8.png)

5. Desaponte o **nome do ambiente** e selecione a **subscrição Azure** para o ponto final da Nuvem Azure.

      ![Selecione a subscrição Azure para o ponto final da Azure Cloud nos serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image9.png)

6. No **nome do serviço app,** desaprote o nome de serviço de aplicação Azure necessário.

      ![Definir nome de serviço de aplicativo Azure em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Insira "Hosted VS2017" na **fila do Agente** para o ambiente hospedado na nuvem Azure.

      ![Definir a fila do agente para o ambiente hospedado na nuvem Azure DevOps](media/solution-deployment-guide-geo-distributed/image11.png)

8. No menu Implementar o Serviço de Aplicações Azure, selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a **localização da pasta**.
  
      ![Selecione pacote ou pasta para o ambiente do Serviço de Aplicações Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selecione pacote ou pasta para o ambiente do Serviço de Aplicações Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image13.png)

9. Guarde todas as alterações e volte a **lançar o gasoduto**.

    ![Economize alterações no oleoduto de lançamento nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image14.png)

10. Adicione um novo artefacto selecionando a construção para a aplicação Azure Stack Hub.

    ![Adicione novo artefacto para app Azure Stack Hub em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Adicione mais um ambiente aplicando a Implementação do Serviço de Aplicações Azure.

    ![Adicionar ambiente à implementação do Serviço de Aplicações Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image16.png)

12. Nomeie o novo ambiente Azure Stack Hub.

    ![Ambiente de nome na implementação do serviço de aplicações Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image17.png)

13. Encontre o ambiente Azure Stack Hub no **separador Tarefa.**

    ![Ambiente Azure Stack Hub em Serviços Azure DevOps em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image18.png)

14. Selecione a subscrição para o ponto final do Azure Stack Hub.

    ![Selecione a subscrição do ponto final do Azure Stack Hub nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image19.png)

15. Desaprote o nome da aplicação web Azure Stack Hub como o nome do serviço de aplicações da App.

    ![Definir nome de aplicativo web Azure Stack Hub em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Selecione o agente Azure Stack Hub.

    ![Selecione o agente Azure Stack Hub nos serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image21.png)

17. Na secção 'Implementar serviço de aplicações Azure', selecione o **Pacote ou Pasta** válido para o ambiente. Selecione **OK** para a localização da pasta.

    ![Selecione pasta para implantação do serviço de aplicações Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selecione pasta para implantação do serviço de aplicações Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image23.png)

18. No separador Variável adicione uma variável `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` nomeada, definir o seu valor como **verdadeiro**, e âmbito para Azure Stack Hub.

    ![Adicionar variável à implementação de aplicativos Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image24.png)

19. Selecione o ícone de disparo de implementação **contínua** em ambos os artefactos e ative o gatilho de implementação **Continua.**

    ![Selecione o gatilho de implementação contínua nos serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image25.png)

20. Selecione o ícone de condições **de pré-implantação** no ambiente Azure Stack Hub e desaccione o gatilho para **depois do lançamento.**

    ![Selecione condições de pré-implantação nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image26.png)

21. Guarde todas as alterações.

> [!Note]  
> Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis ambientais](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição de libertação a partir de um modelo. Estas definições não podem ser modificadas nas definições de tarefa; em vez disso, o item ambiente parental deve ser selecionado para editar estas definições.

## <a name="part-2-update-web-app-options"></a>Parte 2: Atualizar as opções de aplicações web

O [Serviço de Aplicações do Azure](/azure/app-service/overview) oferece um serviço de alojamento na Web altamente dimensionável e com correção automática.

![Serviço de Aplicações do Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Mapeie um nome DNS personalizado existente para Azure Web Apps.
> - Utilize um **registo CNAME** e um **registo A** para mapear um nome DNS personalizado para o Serviço de Aplicações.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Mapear um nome DNS personalizado já existente para as Aplicações Web do Azure

> [!Note]  
> Utilize um CNAME para todos os nomes DNS personalizados, exceto um domínio raiz (por exemplo, northwind.com).

Para migrar um site em direto e o respetivo nome de domínio DNS para o Serviço de Aplicações, veja [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain) (Migrar um nome DNS ativo para o Serviço de Aplicações do Azure).

### <a name="prerequisites"></a>Pré-requisitos

Para completar esta solução:

- [Crie uma aplicação de Serviço de Aplicações,](/azure/app-service/)ou use uma aplicação criada para outra solução.

- Compre um nome de domínio e garanta o acesso ao registo DNS para o fornecedor de domínio.

Atualize o ficheiro da zona DNS para o domínio. A Azure AD verificará a propriedade do nome de domínio personalizado. Utilize [o Azure DNS](/azure/dns/dns-getstarted-portal) para registos DNS Azure/Office 365/external DNS dentro do Azure, ou adicione a entrada de DNS [num registo DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

- Registe um domínio personalizado com um registo público.

- Inicie sessão na entidade de registo de nome de domínio para o domínio. (Um administrador aprovado pode ser necessário para fazer atualizações de DNS.)

- Atualize o ficheiro da zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.

Por exemplo, para adicionar entradas de DNS para northwindcloud.com e www \. northwindcloud.com, configurar as definições de DNS para o domínio raiz northwindcloud.com.

> [!Note]  
> Um nome de domínio pode ser adquirido usando o [portal Azure](/azure/app-service/manage-custom-dns-buy-domain). Para mapear um nome DNS personalizado para uma aplicação Web, o [plano do Serviço de Aplicações](https://azure.microsoft.com/pricing/details/app-service/) dessa aplicação tem de ser um escalão pago (**Partilhado**, **Básico**, **Standard** ou **Premium**).

### <a name="create-and-map-cname-and-a-records"></a>Criar e mapear registos CNAME e A

#### <a name="access-dns-records-with-domain-provider"></a>Aceder a registos DNS com o fornecedor de domínios

> [!Note]  
>  Utilize o Azure DNS para configurar um nome DNS personalizado para aplicações Web Azure. Para obter mais informações, veja [Utilizar o DNS do Azure para oferecer definições de domínio personalizado para um serviço do Azure](/azure/dns/dns-custom-domain).

1. Inscreva-se no site do fornecedor principal.

2. Localize a página para gerir os registos DNS. Cada fornecedor de domínio tem a sua própria interface de registos DNS. Procure áreas do site com os nomes **Nome de Domínio**, **DNS** ou **Gestão de Servidor de Nomes**.

A página de registos DNS pode ser visualizada nos **meus domínios**. Encontre o **link**denominado ficheiro Zone , **DNS Records**ou **configuração Avançada**.

A captura de ecrã seguinte mostra um exemplo de uma página de registos DNS:

![Página de registos DNS de exemplo](media/solution-deployment-guide-geo-distributed/image28.png)

1. No Registo de Nomes de Domínio, **selecione Adicionar ou Criar** para criar um registo. Alguns fornecedores têm ligações diferentes para adicionar diferentes tipos de registos. Consulte a documentação do fornecedor.

2. Adicione um registo CNAME para mapear um subdomínio ao nome de anfitrião padrão da aplicação.

   Para o \. exemplo de domínio www northwindcloud.com, adicione um registo CNAME que mapeie o nome para `<app_name>.azurewebsites.net` .

Depois de adicionar o CNAME, a página de registos DNS parece ser o seguinte exemplo:

![Navegação do portal para a aplicação do Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Ativar o mapeamento de registos CNAME no Azure

1. Num novo separador, inscreva-se no portal Azure.

2. Aceda a Serviços Aplicacionais.

3. Selecione aplicativo web.

4. No painel de navegação esquerdo da página da aplicação no portal do Azure, selecione **Domínios personalizados**.

5. Selecione o **+** ícone ao lado do nome de **anfitrião.**

6. Digite o nome de domínio totalmente qualificado, como `www.northwindcloud.com` .

7. Selecione **Validar**.

8. Se indicado, adicione registos adicionais de outros tipos `A` (ou `TXT` ) aos registos DNS dos registos de nome de domínio. A Azure fornecerá os valores e tipos destes registos:

   a.  Um registo **A**, para mapear o endereço IP da aplicação.

   b.  Um registo **TXT**, para mapear para o nome de anfitrião predefinido da aplicação, `<app_name>.azurewebsites.net`. O Serviço de Aplicações utiliza este registo apenas no momento da configuração para verificar a propriedade personalizada do domínio. Após verificação, elimine o registo TXT.

9. Complete esta tarefa no separador de registo de domínio e revalida até que o botão **de nome de anfitrião Add** seja ativado.

10. Certifique-se de que **o tipo de registo hostname** está definido para **CNAME** (www.example.com ou qualquer subdomínio).

11. Selecione **Adicionar nome de anfitrião**.

12. Digite o nome de domínio totalmente qualificado, como `northwindcloud.com` .

13. Selecione **Validar**. O **Add** é ativado.

14. Certifique-se de que **o tipo de gravação do hostname** está definido para um **recorde** (example.com).

15. **Adicione o nome de anfitrião**.

    Pode levar algum tempo para que os novos anfitriões sejam refletidos na página de **domínios personalizados** da aplicação. Experimente atualizar o browser para atualizar os dados.
  
    ![Domínios personalizados](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Se houver um erro, aparecerá uma notificação de erro de verificação na parte inferior da página. ![Erro de verificação de domínio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  Os passos acima podem ser repetidos para mapear um domínio wildcard \* (.northwindcloud.com). Isto permite a adição de quaisquer subdomínios adicionais a este serviço de aplicações sem ter de criar um registo CNAME separado para cada um. Siga as instruções do registo para configurar esta definição.

#### <a name="test-in-a-browser"></a>Teste num browser

Navegue pelo(s) nome(s) DNS configurado anteriormente (por exemplo, `northwindcloud.com` ou `www.northwindcloud.com` .

## <a name="part-3-bind-a-custom-ssl-cert"></a>Parte 3: Bind a custom SSL cert

Nesta parte, iremos:

> [!div class="checklist"]
> - Ligue o certificado SSL personalizado ao Serviço de Aplicações.
> - Imponha HTTPS para a aplicação.
> - Automatizar a ligação do certificado SSL com scripts.

> [!Note]  
> Se necessário, obtenha um certificado SSL de cliente no portal Azure e ligue-o à aplicação web. Para mais informações, consulte o [tutorial dos Certificados de Serviço de Aplicações.](/azure/app-service/web-sites-purchase-ssl-web-site)

### <a name="prerequisites"></a>Pré-requisitos

Para completar esta solução:

- [Crie uma aplicação de Serviço de Aplicações.](/azure/app-service/)
- [Mapeie um nome DNS personalizado para a sua aplicação web.](/azure/app-service/app-service-web-tutorial-custom-domain)
- Adquira um certificado SSL a uma autoridade de certificados fidedignos e utilize a chave para assinar o pedido.

### <a name="requirements-for-your-ssl-certificate"></a>Requisitos do certificado SSL

Para utilizar um certificado no Serviço de Aplicações, aquele tem de cumprir os seguintes requisitos:

- Assinado por uma autoridade de certificado de confiança.

- Exportado como um ficheiro PFX protegido por palavra-passe.

- Contém chave privada com pelo menos 2048 bits de comprimento.

- Contém todos os certificados intermédios na cadeia de certificados.

> [!Note]  
> **Os certificados de Criptografia da Curva Elíptica (ECC)** funcionam com o Serviço de Aplicações, mas não estão incluídos neste guia. Consulte uma autoridade de certificação para assistência na criação de certificados ECC.

#### <a name="prepare-the-web-app"></a>Preparar a aplicação web

Para vincular um certificado SSL personalizado à aplicação web, o [plano de Serviço de Aplicações](https://azure.microsoft.com/pricing/details/app-service/) deve estar no nível **Básico,** **Standard**ou **Premium.**

#### <a name="sign-in-to-azure"></a>Iniciar sessão no Azure

1. Abra o [portal Azure](https://portal.azure.com/) e vá para a aplicação web.

2. A partir do menu esquerdo, selecione **Serviços de Aplicações**e, em seguida, selecione o nome da aplicação web.

![Selecione aplicativo web no portal Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Verificar o escalão de preço

1. Na navegação à esquerda da página da aplicação web, percorra a secção **Definições** e selecione **Scale up (Plano de Serviço de Aplicações)**.

    ![Menu de escala na aplicação web](media/solution-deployment-guide-geo-distributed/image34.png)

1. Certifique-se de que a aplicação web não está no nível **Gratuito** ou **Partilhado.** O nível atual da aplicação web é destacado numa caixa azul escura.

    ![Verifique o nível de preços na aplicação web](media/solution-deployment-guide-geo-distributed/image35.png)

O SSL personalizado não é suportado no nível **Gratuito** ou **Partilhado.** Para aumentar a escala, siga os passos na secção seguinte ou na página de **nível de preços** e salte para [o Upload e prenda o certificado SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Aumentar verticalmente o seu plano do Serviço de Aplicações

1. Selecione um dos escalões **Básico**, **Standard** ou **Premium**.

2. Selecione **Selecionar**.

![Escolha o nível de preços para a sua aplicação web](media/solution-deployment-guide-geo-distributed/image36.png)

A operação de escala fica completa quando a notificação é apresentada.

![Notificação para “aumentar verticalmente”](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Ligue o seu certificado SSL e funda certificados intermédios

Fundir vários certificados na cadeia.

1. **Abra cada certificado** que recebeu num editor de texto.

2. Criar um ficheiro para o certificado fundido chamado *mergedcertate.crt*. Num editor de texto, copie o conteúdo de cada certificado para este ficheiro. A ordem dos seus certificados deve seguir a ordem na cadeia de certificados, a começar no seu certificado e a terminar no certificado de raiz. O aspeto é igual ao do exemplo abaixo:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Exportar o certificado para PFX

Exportar o certificado SSL fundido com a chave privada gerada pelo certificado.

Um ficheiro chave privado é criado através do OpenSSL. Para exportar o certificado para PFX, executar o seguinte comando e substituir os espaços reservados `<private-key-file>` e pelo caminho chave privado e o ficheiro de certificado `<merged-certificate-file>` fundido:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Quando solicitado, defina uma senha de exportação para o upload do seu certificado SSL para o Serviço de Aplicações mais tarde.

Quando o IIS ou **Certreq.exe** forem utilizados para gerar o pedido de certificado, instale o certificado numa máquina local e, em seguida, [exporte o certificado para PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

#### <a name="upload-the-ssl-certificate"></a>Faça o upload do certificado SSL

1. Selecione **as definições SSL** na navegação à esquerda da aplicação web.

2. Selecione **o Certificado de Upload**.

3. No **Ficheiro de Certificado PFX,** selecione o ficheiro PFX.

4. Na **palavra-passe do Certificado,** digite a palavra-passe criada ao exportar o ficheiro PFX.

5. Selecione **Carregar**.

    ![Certificado SSL de upload](media/solution-deployment-guide-geo-distributed/image38.png)

Quando o Serviço de Aplicações termina o upload do certificado, aparece na página de **definições SSL.**

![Definições de SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Vincular o seu certificado SSL

1. Na secção **de encadernações SSL,** selecione **Adicionar encadernação**.

    > [!Note]  
    >  Se o certificado tiver sido carregado, mas não aparecer em nomes de domínio no dropdown **do Nome Anfitrião,** tente refrescar a página do navegador.

2. Na página **Add SSL Binding,** utilize as descidas para selecionar o nome de domínio para garantir e o certificado a utilizar.

3. Em **Tipo de SSL**, selecione se vai utilizar [**SSL baseado em Indicação do Nome de Servidor (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) ou baseado em IP.

    - **SSL baseado em SNI:** Podem ser adicionadas ligações SSL baseadas em SNI múltiplas. Esta opção permite utilizar vários certificados SSL para proteger múltiplos domínios no mesmo endereço IP. Os browsers mais modernos (incluindo o Internet Explorer, o Chrome, o Firefox e o Opera) suportam SNI (encontre informações mais abrangentes sobre o suporte de browsers em [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication) [Indicação do Nome de Servidor]).

    - **SSL baseado em IP**: Apenas uma ligação SSL baseada em IP pode ser adicionada. Esta opção permite utilizar apenas um certificado SSL para proteger um endereço IP público dedicado. Para proteger vários domínios, fixe-os todos utilizando o mesmo certificado SSL. O SSL baseado em IP é a opção tradicional para a ligação SSL.

4. Selecione **Adicionar Ligação**.

    ![Adicionar ligação SSL](media/solution-deployment-guide-geo-distributed/image40.png)

Quando o Serviço de Aplicações termina o upload do certificado, aparece nas secções **de encadernações SSL.**

![As ligações SSL terminaram o upload](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Remapia o recorde A para IP SSL

Se o SSL baseado em IP não for utilizado na aplicação web, salte para [testar HTTPS para o seu domínio personalizado](/azure/app-service/app-service-web-tutorial-custom-ssl).

Por padrão, a aplicação web utiliza um endereço IP público partilhado. Quando o certificado está ligado ao SSL baseado em IP, o App Service cria um novo e dedicado endereço IP para a aplicação web.

Quando um registo A é mapeado para a aplicação web, o registo de domínio deve ser atualizado com o endereço IP dedicado.

A página **de domínio personalizado** é atualizada com o novo endereço IP dedicado. Copie este [endereço IP](/azure/app-service/app-service-web-tutorial-custom-domain)e, em seguida, remapia o [registo A](/azure/app-service/app-service-web-tutorial-custom-domain) para este novo endereço IP.

#### <a name="test-https"></a>Tester HTTPS

Em diferentes navegadores, vá `https://<your.custom.domain>` para garantir que a aplicação web é servida.

![navegar para aplicativo web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Se ocorrerem erros de validação de certificados, um certificado auto-assinado pode ser a causa, ou os certificados intermédios podem ter sido deixados de fora ao exportar para o ficheiro PFX.

#### <a name="enforce-https"></a>Impor HTTPS

Por predefinição, qualquer pessoa pode aceder à aplicação web utilizando HTTP. Todos os pedidos HTTPS para a porta HTTPS podem ser redirecionados.

Na página da aplicação web, selecione **as definições SL**. Em seguida, em **HTTPS Apenas**, selecione **Ativado**.

![Impor HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

Quando a operação estiver concluída, aceda a qualquer um dos URLs HTTP que apontam para a aplicação. Por exemplo:

- https://<app_name>.azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>Impor TLS 1.1/1.2

A aplicação permite [tLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 por padrão, que já não é considerado seguro pelos padrões da indústria (como [PCI DSS).](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard) Para impor versões do TLS superiores, siga estes passos:

1. Na página da aplicação web, na navegação à esquerda, selecione **as definições SSL**.

2. Na **versão TLS,** selecione a versão TLS mínima.

    ![Impor TLS 1.1 ou 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Criar um perfil do Gestor de Tráfego

1. Selecione Criar um perfil de gestor de tráfego de rede de **recursos**  >  **Networking**  >  **Traffic Manager profile**  >  **Criar**.

2. No painel **Criar perfil do Gestor de Tráfego**, preencha o seguinte:

    1. Em **Nome,** forneça um nome para o perfil. Este nome tem de ser único dentro da zona de manager.net de tráfego e resulta no nome DNS, trafficmanager.net, que é usado para aceder ao perfil de Gestor de Tráfego.

    2. No **método de encaminhamento,** selecione o **método de encaminhamento Geográfico**.

    3. Na **Subscrição**, selecione a subscrição sob a qual criar este perfil.

    4. Em **Grupo de Recursos**, crie um grupo de recursos novo onde colocar este perfil.

    5. Em **Localização do grupo de recursos**, selecione a localização do grupo de recursos. Esta definição refere-se à localização do grupo de recursos e não tem qualquer impacto no perfil do Gestor de Tráfego implantado globalmente.

    6. Selecione **Criar**.

    7. Quando a implantação global do perfil de Gestor de Tráfego está completa, está listado no respetivo grupo de recursos como um dos recursos.

        ![Grupos de recursos na criação do perfil de Gestor de Tráfego](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Adicionar pontos finais do Gestor de Tráfego

1. Na barra de pesquisa do portal, procure o nome de **perfil do Gestor de Tráfego** criado na secção anterior e selecione o perfil do gestor de tráfego nos resultados apresentados.

2. No **perfil de Gestor de Tráfego,** na secção **Definições,** selecione **Pontos de final**.

3. Selecione **Adicionar**.

4. Adicionando o Azure Stack Hub Endpoint.

5. Para **o tipo**, selecione ponto final **externo**.

6. Forneça um **Nome** para este ponto final, idealmente o nome do Azure Stack Hub.

7. Para um nome de domínio totalmente qualificado **(FQDN),** utilize o URL externo para a App Web Azure Stack Hub.

8. Em Geo-mapeamento, selecione uma região/continente onde o recurso está localizado. Por exemplo, **a Europa.**

9. Sob a queda país/região que aparece, selecione o país que se aplica a este ponto final. Por exemplo, **a Alemanha.**

10. Mantenha a caixa **Adicionar como desativado** desmarcada.

11. Selecione **OK**.

12. Adicionar o Azure Endpoint:

    1. Para **tipo**, selecione **Azure endpoint**.

    2. Forneça um **nome** para o ponto final.

    3. Para **o tipo de recurso Alvo**, selecione App **Service**.

    4. Para **o recurso Target**, **selecione Escolha um serviço de aplicações** para mostrar a listagem das Aplicações Web na mesma subscrição. Em **Recurso,** escolha o serviço app utilizado como primeiro ponto final.

13. Em Geo-mapeamento, selecione uma região/continente onde o recurso está localizado. Por exemplo, **América do Norte/América Central/Caraíbas.**

14. Sob a queda país/região que aparece, deixe este lugar em branco para selecionar todos os agrupamentos regionais acima.

15. Mantenha a caixa **Adicionar como desativado** desmarcada.

16. Selecione **OK**.

    > [!Note]  
    >  Crie pelo menos um ponto final com um âmbito geográfico de All (World) para servir como o ponto final padrão para o recurso.

17. Quando a adição de ambos os pontos finais está completa, são exibidas no **perfil de Gestor de Tráfego,** juntamente com o seu estado de monitorização como **Online**.

    ![Estado do ponto final do gestor de tráfego](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>Global Enterprise conta com capacidades de geo-distribuição da Azure

Direcionar o tráfego de dados através do Azure Traffic Manager e os pontos finais específicos da geografia permite que as empresas globais cumpram as normas regionais e mantenham os dados conformes e seguros, o que é crucial para o sucesso de locais de negócios locais e remotos.

## <a name="next-steps"></a>Passos seguintes

- Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)
