# Utilizando AI Search para indexação e consulta de dados

Este projeto demonstra como construir um índice do Azure AI search utilizando dados extraídos de avaliações de clientes.

## Processo

1. Criar um recurso Azure AI Search
2. Criar um recurso Azure AI services
3. Criar uma Storage account
4. Anexar documentos ao Azure Storage
5. Indexar os documentos
6. Consultar o índice
7. Revisar o knowledge store

## Criar um recurso Azure AI Search

1. Realize o login no [Azure portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true)
2. Clique no botão **+ Create a resource**, pesquise por *Azure AI Search*, e crie um recurso **Azure AI Search** com as seguintes configurações:

    - **Subscription**: *Sua subscrição Azure*
    - **Resource group**: *Selectione ou crie um grupo de recursos com um nome único.*
    - **Location/Region**: *Escolha qualquer região disponível*
    - **Pricing tier**: Basic

3. Selecione **Review + create**, e depois você verá a resposta **Validation Success**, selecione **Create**
4. Após a implantação completar, selectione **Go to resource**. Na página de visão geral do Azure AI Search, você pode adicionar índices, importar dados, e pesquisar indíces criados.

## Criar um recurso Azure AI services

Você vai precisar provisionar um recurso **Azure AI services** que seja na mesma localização do seu recurso Azure AI Search. Sua solução do AI Search utilizará esse recurso para enriquecesr os dados na datastore com insights gerados pela AI.

1. Retorne para a pãgina inicial do portal do Azure. Clique no botão **+ Create a resource** e pesquise por *Azure AI services*. Selecione **create** um plano **Azure AI services**. Você será direcionado para uma página de criação do recurso Azure AI services. Configure ele com as seguintes configurações:

    - **Subscription**: *Sua subscrição Azure*
    - **Resource group**:*O mesmo groupo de recursos que o recurso do Azure AI Search*.
    - **Region**: *A mesma localização que o seu recurso Azure AI Search*
    - **Name**: *Um nome único*
    - **Pricing tier**: Standard S0
    - **By checking this box I acknowledge that I have read and understood all the terms below**: Selecionado

2. Selecione **Review + create**. Depois você verá a resposta **Validation Passed**, selecione **Create**
3. Aguarde até a implantação completar, e então veja seus detalhes.

## Criar uma storage account

1. Retorne para a página inicial do portal Azure, e então selecione o botão **+ Create a resource**.
2. Pesquise por *storage account*, e crie um recurso **Storage account** com as seguintes configurações:

    - **Subscription**: *Sua subscrição Azure*
    - **Resource group**: *O mesmo grupo de recurso dos outros recursos criados*
    - **Storage account name**: *Um nome único*
    - **Performance**: Standard
    - **Redundancy**: Locally redundant storage (LRS)

## Anexar documentos ao Azure Storage

1. No painel de controle á esquerda, selecione **Containers**
2. Selecione **+ Container**. Um painel do lado direito abrirá.
3. Informe as seguintes configurações, e clique **Create**

    - **Name**: coffee-reviews
    - **Public access level**: Container (acesso de leitura anônimo para containers e blobs)
    - **Advanced**: *sem mudanças*

4. Em uma nova aba do navegador, faça o download do arquivo *zipped coffee reviews* do link [https://aka.ms/mslearn-coffee-reviews](https://aka.ms/mslearn-coffee-reviews), e extraia os arquivos para a pasta *reviews*
5. No portal Azure, selecione seu container *coffee-reviews*. No container, selecione **Upload**
6. No painel **Upload blob**, selecione **Select a file**
7. Na janela do explorador de arquivos, selecione **todos** arquivos da pasta *reviews*, selecione **Open**, e então clique em **Upload**
8. Depois que o upload se completar, você pode fechar o painel **Upload blob**. Seus documentos agora estão no seu container de armazenamento *coffee-reviews*

## Indexar os documentos

Depois que seus documentos estiverem no armazenamento, você pode utilizar o Azure AI Search para extrair insights dos documentos. O portal Azure provê um *Import data wizard*. Com este wizard, você pode automaticamente criar um índice e indexador para as fontes de dados suportadas. Você usará o wizard para criar um índice, e importar seus documentos de pesquisa do armazenamento para o índice do Azure AI Search.

1. No portal Azure, pesquise seu recurso do Azure AI Search. Na página **Ovewview**, selecione **Import data**
2. Na página **Connect to your data**, na lista **Data Source**, selecione **Azure Blob Storage**. Complete os detalhes do data store com os seguintes valores:

    - **Data Source**: Azure Blob Storage
    - **Data source name**: coffee-customer-data
    - **Data to extract**: Content and metadata
    - **Parsing mode**: Default
    - **Connection string**: * Selecione **Choose an existing connection**. Selecinoe sua conta de armazenamento, selecione o container **coffee-reviews**, e então clique em **Select**
    - **Managed identity authentication**: Nenhuma
    - **Container name**: *esta configuração é autopopulada após você escolher uma conexão existente*
    - **Blob folder**: *Deixe em branco*
    - **Description**: Reviews for Fourth Coffee shops

3. Selecione **Next: Add cognitive skills (Optional)**
4. Na seção **Attach Cognitive Services**, selecione o recurso do Azure AI services.
5. Na seção **Add enrichments**:

    - Mude o **Skillset name** para **coffee-skillset**.
    - Marque a caixa de seleção **Enable OCR and merge all text into merged_content**
    - Certifique-se de que o campo **Source data field** está com o valor **merged_content**
    - Mude o **Enrichment granularity level** para **Pages (5000 character chunks)**
    - Não selecione a opção *Enable incremental enrichment*
    - Selecione os seguintes campos de enriquecimento:

| Cognitive Skill   | Parameter | Field name |
| :---------------- | :------ | :---- |
| Extract location names        |      | locations |
| Extract key phrases           |      | keyphrases |
| Detect sentiment   |     | sentiment |
| Generate tags from images |     | imageTags |
| Generate captions from images | | imageCaption|

6. Em **Save enrichments to a knowledge store**, selecione:

    - Image projections
    - Documents
    - Pages
    - Key phrases
    - Entities
    - Image details
    - Image references

7. Selecione **Azure blob projections: Document**. Uma configuração para *Container name* com o container *knwoledge-store* autopopulado é mostrado. Não mude o nome do container.
8. Selecione **Next: Customize target index**. Mude o **Index name** para **coffee-index**.
9. Certifique-se de que **Key** esteja com o valor **metadata_storage_path**. Deixe vazio o campo **Suggester name** e **Search mode** autopopulado.
10. Revise as configurações padrões dos campos de índices. Selecione **filterable** para todos os campos que já vieram selecionados por padrão.
11. Selecione **Next: Create an indexer**
12. Mude o **Indexer name** para **coffee-indexer**
13. Deixe o **Schedule** como **One**
14. Expanda as **Advanced options**. Certifique-se de que a opção **Base-64 Encode Keys** esteja selecionada, já que chaves indexadas podem deixar o índice mais eficiente.
15. Selecione **Submit** para criar a fonte de dados, skillset, índice e indexador. O indexador é executado automaticamente e executa o pipeline de indexação, que:

    - Extrai os campos de metadados e conteúdo da fonte de dados.
    - Executa o conjunto de habilidades cognitivas para gerar campos mais enriquecidos.
    - Mapeia os campos extraídos para o índice.

16. Retorne para a página do recurso do Azure AI Searcch. No painel à esquerda, em **Search Management**, selecione **Indexers**. Selecione o indexador recém-criado **coffee-indexer**. Espere um minuto, e selecione **Refresh** até o indicador do **Status** indique sucesso.
17. Selecione o nome do indexador para visualizar mais detalhes.

## Consultar o índice

Use o *Search explorer* para escrever e testar consultas. **Search explorer** é uma ferramenta construída no portal Azure que fornece uma forma fácil de validar a qualidade do seu índice de pesquisa. Você pode usar o Search explorer para escrever consultas e revisar os resultados em JSON.

1. Na página de *Overview* do seu serviço *AI Search*, selecione **Search explorer** no topo da tela.
2. Percebe como o índice selecionado é o *coffee-index* que você criou. Abaixo do índice selecionado, mude a *view* para **JSON view**
3. No campo **JSON query editor**, copie e cole:

    ```json
    {
        "search": "*",
        "count": true
    }
    ```

4. Selecione **Search**. A consulta retorna todos os documentos no índice de pesquisa, incluindo um contador com todos os documentos no campo **@odata.count**. A pesquisa do índice deve retornar um documento JSON contendo seus resultados.
5. Agora vamos filtrar por localização. No campo **JSON query editor**, copie e cole:

    ```json
    {
    "search": "locations:'Chicago'",
    "count": true
    }
    ```

6. Selecione **Search**. A consulta pesquisa todos os documentos no índice e filtra por avaliações com a localização de Chicago. Você deve ver ==3== no campo ==@odata.count==
7. Agora vamos filtrar por sentimento. No campo **JSON query editor**, copie e cole:

    ```json
    {
    "search": "sentiment:'negative'",
    "count": true
    }
    ```

8. Clique em **Search**. A consulta pesquisa todos os documentos no índice e filtra por avaliações com um sentimento negativo. Você deverá ver ==1== no campo ==@odata.count==.

## Revisar o knowledge store

Vamos ver o poder da knowlege sotre em ação. Quando você executou o *Import data wizard*, você também criou uma knowledge store. Dentro da knowledge store, você encontrará dados extraídos enriquecidos por habilidades de AI na fomra de projeções e tabelas.

1. No portal Azure, navegue até a sua conta de armazenamento Azure.
2. No menu do painel esquerdo, selecione **Containers**. Selecione o container **knowledge-store**
3. Selecione qualquer um dos itens, e clique no arquivo **objectprojection.json**
4. Selecione **Edit** para ver o JSON produzido para um dos documentos da sua data store Azure.
5. Selecione a localização atual do blob de armazenamento no canto superior esquerdo da tela para retornar aos contêineres da conta de armazenamento.
6. Em *Containers*, selecione o container *coffee-skillset-image-projection*. Selecione qualquer um dos itens.
7. Selecione qualquer um dos arquivos *.jpg*. Selecione **Edit** para ver a imagem armazenada do documento. Perceba como todas as imagens dos documentos sâo armazenadas desta forma.
8. Selecione a localização atual do blob de armazenamento no canto superior esquerdo da tela para retornar aos contêineres da conta de armazenamento.
9. Selecione **Storage browser** no painel á esquerda, e selecione **Tables**. Tem uma tabela para cada entidade do índice. Selecione a tabela *coffeeSkillsetKeyPhrases*

Olhe para as frases chave da knowledge store foi capaz de capturar do conteúdo das avaliações. Muitos dos campos são chaves, para que você possa conectar as tabelas como um banco de dados relacional. O último campo mostra as frases chaves que foram extraídas pelo skillset.