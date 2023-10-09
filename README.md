# Documentação de Uso do BackstopJS

Criação de testes de regressão visual com BackstopJS
## Pré-Requisitos

1. NodeJS (npm) 
2. BackstopJS
3. Leitor de código (vscode)
4. HTML e CSS com id

## Primeiros Passos

1. Inicialização de um projeto NodeJS
2. Inicialização do projeto de testes de regressão visual
3. Documentação oficial da ferramenta

### Inicialização de um projeto NodeJS

- No repositório: npm init -y

### Inicialização de um projeto de testes de regressão visual

#### Baixar o backstopJS
    - npm install backstopjs --save-dev
    - obs: desativar proxy
#### Inicializar o BackstopJS
- npx backstop init

### Documentação oficial da ferramenta

- https://github.com/garris/BackstopJS


## Arquivo backstop.json

1. Id
- Nome da aplicação a testada
2. viewports
- Testa responsividade
    - Em Celulares:
              
        ```
            {
                "label": "phone",
                "width": 320, 
                "height": 480 
            },
        ```
    - Em Tablets:

        ```
            {
                "label": "tablet",
                "width": 1024, 
                "height": 768 
            }
        ```
    * Width: dimensão de largura 
    * Height: dimensão de altura

### Propriedade: scenarios 
- Documentação: 
    - Em Advanced Scenarios
    - https://github.com/garris/BackstopJS#advanced-scenarios
- No arquivo backstopjson
    - **label**
        - Nome um comportamento ou fluxo do teste: 
            - ex: Initial State, 
    - **url**
        - Url do site onde o teste será rodado
    - **misMatchThreshold**
        - Limite de pixels: *quantidade percentual* de diferenças em pixels permitidas para passar no teste
        - No caso de pixel perfect, deve ser 0
    - **requireSameDimensions**
        - Se *true* qualquer mudança no tamanho do seletor disparará falha no teste
    - **clickSelector**   
        - Identifica e clica em um elemento através de seu id
            - Caso o clique do elemento funcione visualmente, isto é, o elemento apareça clicado, o teste passará
    - **clickSelectors**
        - Disponível somente com Puppeteer, driver padrão do BackstopJS 
        - Array de elementos (ids)
    - **onReadyScript**
        - Utilizado para rodar um script puppeteer customizado
    - **select()** 
        - Custom: No backtopJS não há comando que seleciona um campo select. 
            - Necessita criar um script customizado para o puppeteer
        - Seleciona um id específico em um campo    
    - **type()**
        - Digita um texto específico em um campo
#### Primeiro Teste
- **Screenshot: aprovação visual**
    - Compara uma imagem de uma url com uma outra imagem de uma mesma ou de outra url
        - *Caso queira se certificar que a formatação de um site permanece a mesma após alterações no backend, por exemplo, você pode:*
            1. Rodar o teste antes da alteração e aprovar
            2. Rodar novamente após alteração e ver se o teste passou ou falhou
        - *Caso queira se certificar que o site segue o layout você pode:*
            1. Passar a URL da imagem de layout aberta no navegador
            2. Rodar o teste e aprovar
            3. Passar a URL do layout implementado (por exemplo, no staging ou ambiente dev)
            4. Rodar o teste, se o teste falhar, o layout implementado está diferente do layout desenhado       
    - Propriedades do cenário: label, URL, misMatchThreshold, requireSameDimensions
    
    ```
    "scenarios": [
        {
        "label": "Initial State",
        "url": "https://ticketbox-backstopjs-tat.s3.eu-central-1.amazonaws.com/index.html",
        "misMatchThreshold" : 0,
        "requireSameDimensions": true
        }
    ],    
    ```
    - Para o armazenamento da primeira imagem:
        1. Rodar o teste:
        - npx backstop test
        - O primeiro teste falhará: porque não há referência para comparar a imagem
        - Ao rodar um teste, uma pasta com o relatório do teste é criada em backstop_data, chamada bitmaps_test
        2. Aprovar a screenshot
        - npx backstop approve
        - Aprovará a imagem: agora ela será a imagem de referência
        - Será criada uma pasta chamada bitmaps_reference onde salvará a imagem da URL do teste realizado
    - Para comparação com uma     
        3. Rodar novamente o teste: 
        
        -> Caso seja uma mesma URL, não alterar URL
        
        -> Caso seja uma outra URL, altere a URL antes de rodar
        - Caso não haja diferença visual, isto é o layout da interface da URL seja igual o layout da interface do primeiro teste (que teve a imagem aprovada), o teste passará
        - caso haja diferença visual, o teste falhará

#### Segundo Teste
- **Clicar em um elemento em campo radius button**
- Propriedades do cenário: label, URL, misMatchThreshold, requireSameDimensions, clickSelector
- No exemplo abaixo, a id do checkbox clicado é #vip 
```
    {
      "label": "Checar a opção VIP",
      "url": "https://ticketbox-backstopjs-tat.s3.eu-central-1.amazonaws.com/index.html",
      "misMatchThreshold" : 0,
      "requireSameDimensions": true,
      "clickSelector": "#vip"
    }
```
- Deve repetir o fluxo de armazenamento da imagem realizado no **Primeiro Teste**
### Terceiro Teste
- **Clicando três elementos de um campo checkbox**
- Propriedades do cenário: label, URL, misMatchThreshold, requireSameDimensions, clickSelectors
- No exemplo abaixo, as ids são passadas dentro de um array: #friends, #publication, #social-media
```
    {
      "label": "Checar opções de seleção múltipla",
      "url": "https://ticketbox-backstopjs-tat.s3.eu-central-1.amazonaws.com/index.html",
      "clickSelectors":["#friend", "#publication", "#social-media"],
      "misMatchThreshold" : 0,
      "requireSameDimensions": true
    }
```

#### Quarto Teste
**Selecionando três elementos em um campo select**
- Na pasta /backstop_data/puppet criar uma nova pasta custom, criar um arquivo chamado selectTicket.js
- Colocar o script do select utilizando a linguagem do puppeteer onde #ticket-quantity é o id do campo select
```
module.exports = async page => {
    await page.select("#ticket-quantity", "3") 
}
```
- Em scenarios utilizar as propriedades: label, URL, misMatchThreshold, requireSameDimensions, onReadyScript
- Em "onReadyScript" colocar o caminho do script customizado
```
    {
      "label": "Selecionar três tickets",
      "url": "https://ticketbox-backstopjs-tat.s3.eu-central-1.amazonaws.com/index.html",
      "onReadyScript": "puppet/custom/selectTickets.js",
      "misMatchThreshold" : 0,
      "requireSameDimensions": true
    }   
```

#### Quinto Teste
**Digitando email inválido**
- Na pasta /backstop_data/puppet/custom criar um arquivo chamado typeInvalidEmail.js
- Colocar o script do select utilizando a linguagem do puppeteer onde #email é o id do campo de texto do email
- Esse teste prevê que o campo de email terá máscara de email
```
module.exports = async page => {
    await page.type("#email", "invalid-email.com"); 
}
```
- Em scenarios utilizar as propriedades: label, URL, misMatchThreshold, requireSameDimensions, onReadyScript
- Em "onReadyScript" colocar o caminho do script customizado
```
    {
      "label": "Email inválido",
      "url": "https://ticketbox-backstopjs-tat.s3.eu-central-1.amazonaws.com/index.html",
      "onReadyScript": "puppet/custom/typeInvalidEmail.js",
      "misMatchThreshold" : 0,
      "requireSameDimensions": true
    }    
```