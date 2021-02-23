# Testando A Performance De Tempo De Execução De Um App React Native Usando Hermes

### Depure como um detetive. Este artigo resume como o React Native funciona e da uma visão geral da performance do tempo de execução (Runtime) do applicativo usando Hermes.

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f88b1388c10d6ec6f029b27_header_img.png)

Temos uma ferramenta para ajudá-lo a resolver esses mistérios com mais rapidez, para que você possa pendurar seu chapéu _Sherlock Holmes_ e voltar a escrever o código.

[Hermes-profile-transformer](https://www.npmjs.com/package/hermes-profile-transformer) permite que você visualize o desempenho do seu aplicativo de uma maneira fácil e precisa.


Antes de mergulhar no que a ferramenta faz, vamos ter certeza de que você sabe o que é _criação de perfil_ quando se trata de aplicativos.  [Criação de Perfil ou _Profiling_](https://en.wikipedia.org/wiki/Profiling_(computer_programming))  é "uma forma de análise dinâmica de programa que mede, por exemplo, o espaço (memória) ou complexidade de tempo de um programa, o uso de instruções específicas ou a frequência e duração das chamadas de função."

Simplificando - o perfil gerado é extremamente importante para entender o desempenho do tempo de execução do seu programa, ou em particular do aplicativo React Native, e encontrar soluções para otimizar esse tempo de execução.

O Hermes-profile-transformer ajuda os desenvolvedores a criar o perfil e visualizar o desempenho do JavaScript em execução no [Hermes](https://github.com/facebook/hermes) em um aplicativo React Native. Uma maneira simples de analisar o desempenho de um programa é analisar o perfil de amostragem na guia [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools).

É aqui que nos deparamos com um obstáculo. O criador de perfil do Hermes gera um formato diferente dos processos do Chrome DevTools. Esses dois formatos são como fogo e gelo. É difícil fazer com que eles formem um bom par.

É aí que entra nossa ferramenta. Ela transforma a saída do criador de perfil Hermes em um formato que o rastreamento de eventos do Chrome pode manipular. Isso permite que os desenvolvedores usem o Chrome DevTools para visualizar o perfil de amostragem de seu aplicativo e entender quais funções estão demorando muito para serem concluídas.

Vamos mergulhar na criação de perfil de seu aplicativo React Native com Hermes e visualizar o desempenho do aplicativo.

## **O Que É Hermes?**

Hermes é um JavaScript Engine de código aberto projetado para executar aplicativos React Native no Android. Hermes se destaca entre seus concorrentes por seu desempenho quando se trata de tempos de inicialização, uso de memória e tamanho do aplicativo.

O uso do Hermes em aplicativos React Native não é necessário. No entanto, considerando que o Hermes foi projetado explicitamente pelo Facebook para o React Native, é provável que seja o modo mais adotado de executar aplicativos do React Native no futuro.

Para obter mais informações sobre o Hermes e como usá-lo, você pode verificar esta [documentação](https://reactnative.dev/docs/hermes).

O Hermes melhora o desempenho do aplicativo React Native em um ambiente mobile. Você pode ver exemplos disso nos experimentos realizados por [Ram](https://twitter.com/nparashuram). Essas demonstrações provaram quão poderoso e rápido Hermes pode ser quando usado no ecossistema React Native.

![Hermes Performance Improvements](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd91206f535d549230_Zkbbd4HAWYI-boTHnnF6qZEJnJEcK3SZbZ1_ki3ygKQkdBUKyHvW9KRTNJsgKFkQHCqXI7xzu5G7cngbw_IvGsSKWGUrRt4zeZS6t9trAOJvkRKGO3UFaelMBX-0cj0cmMcyLPer.jpeg)

Você pode mergulhar mais fundo nos excelentes experimentos de Ram com Hermes [bem aqui](https://blog.nparashuram.com/2019/07/facebook-announced-hermes-new.html).

### Formato Dos Eventos Rastreados 

Nos aplicativos React Native em execução no Hermes possuem um mecanismo para criar o perfil de desempenho do aplicativo conforme ele é executado. O processo para fazer isso é relativamente simples. O menu do desenvolvedor de aplicativos RN em execução no Hermes tem uma opção para alternar a criação de perfil. Ao habilitar a criação de perfil, o criador de perfil começa a criar eventos de rastreamento (na forma de amostras e frames de pilha) que podem ser usados para obter informações de tempo das funções em execução.

No entanto, os eventos rastreados criados pelo criador de perfil Hermes são do formato _JSON Object_, em oposição ao formato _JSON Array_ com suporte do Chrome. Portanto, precisamos de um [transformador](https://www.npmjs.com/package/hermes-profile-transformer) que converta o perfil Hermes em formato _JSON Array_, que é compatível com Chrome DevTools.

### Entendendo Os Eventos Rastreados

Os eventos rastreados são pontos de dados que determinam o estado de um aplicativo em qualquer ponto do tempo. Esses pontos de dados incluem nomes das funções em execução, seus processos e IDs de thread, entre muitos outros. Os perfis de amostragem mais comuns geralmente podem ser analisados como _JSON_ e podem estar em 2 formatos:

**Formato Array JSON**  - Este é o formato mais simples. Ele usa uma série de objetos de evento que indicam os horários de início e fim (com base em fases) e pode ser carregado no Chrome DevTools para visualizar o desempenho de um aplicativo. Por exemplo:

```json
[
{ name: "Asub", cat: "PERF", ph: "B", pid: 22630, tid: 22630, ts: 829 },
{ name: "Asub", cat: "PERF", ph: "E", pid: 22630, tid: 22630, ts: 833 },
];
```

O formato _JSON Array_ é possivelmente a maneira mais simples e eficaz de armazenar as informações de criação de perfil de um aplicativo. É fácil de ler e, portanto, amplamente adotado.

**Formato JSON Object**  - O formato _JSON Object_, como o próprio nome sugere, é uma coleção de pares de valores-chave que podem ser usados para capturar o estado do aplicativo. Por exemplo:

```json
{    "traceEvents": [
     {"name": "Asub", "cat": "PERF", "ph": "B", "pid": 22630, "tid": 22630, "ts": 829},
     {"name": "Asub", "cat": "PERF", "ph": "E", "pid": 22630, "tid": 22630, "ts": 833}
   ],
   "displayTimeUnit": "ns",
   "systemTraceEvents": "SystemTraceData",
   "otherData": {
     "version": "My Application v1.0"
   },
   "stackFrames": {...}
   "samples": [...],
 }
```

O objeto contém 3 chaves, sendo, `traceEvents`,` samples` e `stackFrames`, cada uma delas é extremamente importante para analisar o desempenho do aplicativo. A vantagem dessa estrutura sobre o formato de matriz _JSON_ é que as informações são armazenadas em um formato mais eficiente, permitindo que você adicione pontos de dados adicionais no futuro.

No caso do perfil Hermes, obtemos um objeto _JSON_ com eventos de metadados em sua propriedade `traceEvents` e as propriedades `samples` e `stackFrames` capturando informações sobre todas as outras funções em execução durante o intervalo de criação de perfil. Essas propriedades contêm informações essenciais, como categorias de eventos, fases e carimbos de data / hora, que nos ajudam a reunir muitas informações sobre nosso aplicativo React Native.

### Eventos & Fases

Eventos são ações disparadas enquanto um aplicativo está em execução. Esses eventos podem ser de diferentes tipos e ter várias fases. O tipo mais simples de evento é o Evento de Duração (Duration Events). Como o nome indica, esses eventos simplesmente capturam e registram o tempo que leva para concluir uma ação. Dois pontos de dados distintos, sendo, o `início` e o` fim` de um evento de duração, especificam o status desses eventos. Esses pontos de dados são conhecidos como fases.

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f877cca6a6e5b56035138c3_Screen%20Shot%202020-10-14%20at%203.33.24%20PM.png)

Para nosso caso de uso específico, os eventos apresentados acima são apenas chamadas de função. Amostras diferentes indicam quando uma função é empurrada e retirada da pilha de chamadas. As fases dos eventos são as propriedades mais importantes para rastrear eventos. Porque? Porque eles indicam o estado das funções na pilha (_stack_) de chamadas. Podemos supor que o perfil Hermes consiste apenas em eventos de duração (_Duration Events_).

## **Interpretação De Eventos Diferentes Em Um Perfil**

O perfil Hermes é do formato objeto _JSON_, no entanto, a propriedade `traceEvents` não contém informações de temporização das funções como esperamos. Em vez disso, a propriedade `traceEvents` simplesmente contém certos eventos de metadados que não fornecem informações sobre o desempenho do aplicativo. Essas informações são capturadas nas amostras e nas propriedades `stackFrames`.

Os `samples`, como o nome sugere, consistem em cópias instantânea da função de pilha (_stack_) em uma data/hora especifica. Eles também contêm uma propriedade `sf` que corresponde a um elemento na propriedade `stackFrames` do perfil.

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd6573ac25e439d21a_JUaQ26QqK5jYK6WSeXFFk-MNTIqQhq13YaMJ9R2kOyOAdfuptryD8kX1P4QFwVl0kbLfUJ0o_4fvGFDUtk-ygQqqtRGV0uvheyTKDrdFRkAPtQkYYoHyYPJTi7kHEmIpKA4RDQCv.png)

Os eventos também podem ser classificados em categorias amplas com base em sua origem. Os eventos que geralmente são escritos em código-fonte se enquadram na categoria de Javascript, enquanto os eventos que ocorrem nativamente na plataforma são categorizados apropriadamente como Nativos (_Native_). Metadados são outra categoria primária de eventos e geralmente são coletados na propriedade `traceEvents` do perfil Hermes.

### Definições

1.  _Nodes_ - Os nós representam todos os eventos possíveis em uma pilha (_stack_) de chamadas de função. Por exemplo:

```json
"2" : {
"line": "8",
"column": "12",
"funcLine": "8",
"funcColumn": "12",
"name": "f1(test.js:8:12)",
"category": "JavaScript",
"parent": 1
}
```

Cada um desses objetos de estrutura de pilha (_stack_) pode ter eventos de início e fim correspondentes. Os eventos individuais de início/fim podem ser chamados de _Nodes_. Os _Nodes_ só têm contexto em carimbos de data/hora específicos, ou seja, um _Node_ pode estar no estado `start`,` end` ou `running` em um determinado carimbo de data/hora.

2. Active & Last Active Nodes (Nós ativos e último nós ativos) - Com base na definição anterior, os _Nodes_ ativos em qualquer momento específico são os _Nodes_ que estão ativos no carimbo de data/hora atual. Os Últimos _Nodes_ ativos representam, da mesma forma, os _Nodes_ que estavam ativos no tempo de amostragem anterior.

3.  Start & End Nodes (Nós de começo e fim) - podemos definir _Nodes_ iniciais como _Nodes_ ativos no carimbo de data/hora atual, mas ausentes nos últimos _Nodes_ ativos, enquanto os _Nodes_ finais podem ser definidos como _Nodes_ nos últimos _Nodes_ ativos, mas ausentes nos _Nodes_ ativos atuais.

![Start and end nodes](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd24eace8ffdfe96fe_0ZVYDTKbdwc1rIqnI-vCQ7DK8oTu-m7T9a3czeUPEY7XJKu90975J8irvqL2lH4Y0jgjiY7FLaYOTdegN8hXkQhxuyo0bQzs8vy2hd1r6kWoLDG5UUsBgAqVH_lzDwGyKzO5ucRr.png)

O transformador de perfil Hermes funciona identificando nós inicial e final em cada registro de data e hora e criando eventos a partir desses nós para serem exibidos no Chrome DevTools.

## **Uso: Com React Native CLI**

Também implementamos um novo comando `react-native profile-hermes` no [React Native CLI](https://github.com/react-native-community/cli) para tornar o processo mais suave para os desenvolvedores. O comando transforma automaticamente o perfil usando nosso pacote `hermes-profile-transformer` e puxa o dispositivo convertido para a máquina local do usuário. Siga estas etapas para começar a criar o perfil de seu aplicativo:

**Passo 1**‍

Primeiro, você precisa habilitar o Hermes em seu aplicativo React Native seguindo estas [instruções](https://reactnative.dev/docs/hermes).


**Passo 2**

Registre um gerador de perfil de amostragem Hermes seguindo estas etapas:

- Navegue até o terminal do servidor Metro em execução.
- Pressione d para abrir o Menu do Desenvolvedor.
- Selecione `Enable Sampling Profiler` (Ativar Perfil de Amostragem).
- Execute seu JavaScript em seu aplicativo pressionando qualquer botão.
- Abra o Menu do Desenvolvedor pressionando d novamente.
- Selecione `Disable Sampling Profiler` para parar a gravação e salvar o perfil de amostragem. Um toast mostrará o local onde o criador de perfil de amostragem foi salvo, geralmente em /data/user/0/com.appName/cache/*.cpuprofile

![Toast Notification of Profile saving](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd5e3c8911ac2b99d8_Wyju3SrkXUDjfkgfD1sdmfZbRUN99w94OjnHX9fl4-Fcq01L-bRiLSBQ0ubN7ndaKZK8U8BxsSyKs-Iiw8KtP-3q63ftyM4NhtW7MTsETWg7g6sqDDQOce6IGWLyeok-W4r6cSpb.png)

**Passo 3**

Execute o comando `profile-hermes` da CLI.

Você pode usar o comando `react-native profile-hermes` da React Native CLI para puxar o perfil convertido do Chrome para sua máquina local. Observe que o comando só funciona se o aplicativo for executado no modo de desenvolvimento, pois o comando usa `adb pull` para baixar o perfil do dispositivo Android do usuário.

Vamos mergulhar um pouco mais fundo nas etapas do seu fluxo de desenvolvimento.

Primeiro, você deseja obter source maps (mapas de origem) que ajudarão o perfil a associar eventos de rastreamento ao código do aplicativo. Embora seja opcional, ainda é altamente recomendável que os source maps (mapas de origem) sejam fornecidos para ajudar a fornecer melhores insights sobre o desempenho do aplicativo. Você pode fazer isso simplesmente ativando bundleInDebug se o aplicativo estiver sendo executado no modo de desenvolvimento, assim:

No arquivo `android/app/build.gradle` do seu aplicativo, adicione:

```json
project.ext.react = [
  bundleInDebug: true,
]
```

Certifique-se de limpar a _build_ sempre que fizer qualquer alteração no `build.gradle`

Limpe a _build_ executando:


```bash
cd android && ./gradlew clean
```
Execute seu aplicativo:

```bash
npx react-native run-android
```

Execute o comando para baixar o perfil convertido:

```bash
npx react-native profile-hermes [destinationDir]
```

Você pode ler mais sobre o uso do comando, incluindo os argumentos opcionais que ele leva, na documentação [aqui](https://github.com/react-native-community/cli/blob/master/docs/commands.md#profile-hermes).

**Passo 4**

Abra o perfil baixado no Chrome DevTools.

Para visualizar o perfil baixado na etapa 3 acima no Chrome DevTools, faça o seguinte:

- Abra o Chrome DevTools.
- Selecione a guia Desempenho.
- Clique com o botão direito e escolha Carregar perfil ...

![Loading a performance profile on Chrome DevTools](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd9619074d36c2d175_VCMe9s8czgULbLL5O7Cy_fXEuPzJWOCFGMh6BcScvnp7ckU_i73idCENh0z-ZdxCra_eTOFSNrTShkYrlSgsWfRf9p-ozU3PccO84s-JRNq-AE_EuRsrYoe2RdjoES09_hDzISZ1.png)

Agora você pode visualizar o desempenho do tempo de execução do seu aplicativo observando a frequência e a duração de cada chamada de função. Iremos sugerir alguns insights que obtemos do perfil na próxima parte.

## **Uso: Com Vanilla JavaScript**

Hermes é um Engine de JavaScript, portanto, também pode ser usado para criar perfis de JavaScript Vanilla fora de um aplicativo React Native.

O Hermes pode ser instalado em seu sistema para executar o Vanilla JavaScript usando o pacote [jsvu](https://www.npmjs.com/package/jsvu) no npm. Depois de instalar o Hermes, podemos executar nossos arquivos com o Hermes simplesmente executando

```bash
hermes index.js
```

As vantagens da criação de perfis ainda permanecem, pois podemos identificar implementações eficientes de funções. Para demonstrar isso, podemos criar o perfil de uma função simples para calcular o enésimo número de Fibonacci.

Vamos usar duas abordagens como exemplo. Sabemos com certeza que a implementação recursiva do cálculo do número de Fibonacci é mais lenta do que a implementação da Programação Dinâmica.

Podemos verificar isso definindo o perfil de nosso código Javascript. Simplesmente execute sua função com o sinalizador `--sample-profiling`. Aqui está o comando para fazer exatamente isso:

`hermes --sample-profiling ./index.js 2> trace-hermes.json`

![](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f877cf68fd6da80ffe89644_Screen%20Shot%202020-10-14%20at%203.32.35%20PM.png)

## **Insights De Informações De Criação De Perfil**

A duração de uma chamada de função pode ser identificada a partir dos carimbos de data/hora dos eventos de início e fim correspondentes. Se esta informação for capturada com sucesso, os eventos aparecem em barras horizontais.

![Profile](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd2452dd1c5ac6eb4e_MfjcXmCNV-arfpTl0nd9AS1PqI-OJsDpTXyYbNZDdxd0AjHYcNCEqyvnXhIpFQT58Wx8rnGiCT_INdX8EYbLa93XhzhO1miDkPC_mmjHR1dQa5HE6HLakum-H_jDEIxiLbbISqIX.png)

Conforme mostrado na captura de tela, as funções podem ser vistas em linhas horizontais, e o eixo de tempo horizontal pode nos ajudar a determinar por quanto tempo uma função é executada. As chamadas de função também podem ser selecionadas para detalhes extras:

![EventInfo](https://uploads-ssl.webflow.com/5f64c4e9139e46231d773b0a/5f861bfd70e3920bdc1c6247_FoTEKIPWHrW2pqDzC4qs46-E9R2WVwksnOnf9KihS6gx_P8VeaOqt5QvIBZ7AdGTOoePA0AsYh-YoQEnopNFprijjwYpgiuFRacnHuYBe7zznIsIjGfA9oKSTwMpPdsvPtR1UB7n.jpeg)

Esses resumos contêm explicitamente por quanto tempo a função é executada e também podemos identificar os números de linha e coluna por meio dos quais também podemos isolar partes de nossa base de código que nos deixam mais lentos.

Os source maps (mapas de origem) agregam muito valor aqui. Por padrão, os números de linha e coluna são indicados em arquivos de pacote. No entanto, quando usamos source maps (mapas de origem), podemos mapear nossas chamadas de função para os arquivos "não agrupados" ou código-fonte escrito pelo usuário, melhorando a experiência de depuração.

As categorias de eventos nos ajudam a determinar a cor das linhas da função na visualização. Em geral, temos 2 tipos de categorias:

1. Obtidos a partir de source maps (mapas de origem) - Source maps (mapas de origem) podem ser fornecidos opcionalmente para aumentar as informações fornecidas pelo Hermes. Os source maps (mapas de origem) nos ajudam a identificar melhores categorias de eventos, agregando valor à nossa visualização. Essas categorias incluem duas categorias amplas: 

```
   `react-native-internals`
```

```
   `other_node_modules`
```

2. Obtido de Amostras Hermes - Essas categorias são obtidas por padrão e podem ser mapeadas para chamadas de função. As categorias Javascript e Native são vistas predominantemente e, em conjunto com os source maps (mapas de origem), isso pode nos ajudar a diferenciar o código clichê escrito em node_modules e o código real que escrevemos.

Como exemplo do perfil acima na imagem, podemos identificar algumas funções que têm uma longa duração. Por exemplo, `batchedUpdates $ 1` tem alta frequência e longa duração. É chamado 10 vezes no nível de zoom da imagem e leva tempos diferentes em cada chamada. Podemos, portanto, notar que esta é uma função crucial se quisermos otimizar nosso código para velocidade. (Esta função, no entanto, é interna para `react-native`, tendo a categoria de` react-native-internals`, portanto, saber quando e por quanto tempo esta função é chamada pode ser usado pelos desenvolvedores do núcleo React Native para melhorar o desempenho do React Native) .

Outro exemplo é a função `unstable_runWithPriority` que pertence à categoria` other_node_modules` que se estende por ~ 1 segundos. Notamos que a cor dessa barra em particular também é diferente. Neste caso, é do pacote `scheduler`, que pode ser entendido lendo o resumo da chamada de função no Chrome DevTools. As chamadas de função são codificadas por cores para denotar sua origem. Isso pode nos ajudar a identificar quais funções estão sendo chamadas de pacotes npm e como elas afetam o desempenho de nosso aplicativo.

Esperamos que este artigo tenha fornecido informações úteis sobre como criar o perfil de seu aplicativo React Native usando Hermes e visualizar seu desempenho no Chrome DevTools. O novo comando `react-native profile-hermes` atualmente funciona apenas no modo de desenvolvimento. No entanto, isso pode ser usado na produção também, pois mais detalhes virão em breve.

## Sobre a G2i

‍**G2i une empresas incríveis com desenvolvedores incrivelmente talentosos**

G2i é o _único_ _marketplace_ para desenvolvedores especializados. Nosso foco é encontrar e verificar os desenvolvedores de JavaScript que possuem profundo conhecimento de domínio em uma ou duas áreas. Quando você contrata um desenvolvedor React Native na plataforma G2i, ele terá passado em um teste projetado especificamente para verificar seu conhecimento intrincado dos prós e contras do React Native.

Acreditamos em mergulhar profundamente em um ou dois reservatórios de conhecimento, em vez de arranhar a superfície de múltiplas áreas do conhecimento. Com uma base de conhecimento profunda, os desenvolvedores podem começar a trabalhar imediatamente. Nosso teste decisivo é: "Este desenvolvedor pode causar um impacto em sua base de código na primeira semana?"

Nosso exame é extremamente profundo. Cada desenvolvedor passa por uma verificação cultural, avaliação de idioma, um desafio de código e uma entrevista técnica que são específicas para a pilha de tecnologia com a qual o desenvolvedor deseja trabalhar. Por exemplo, se um desenvolvedor que você contratar deseja trabalhar com o React, ele já terá passado em nosso desafio de codificação específico do React.

Acreditamos na total transparência no processo de contratação. Você receberá perfis técnicos de cada desenvolvedor que incluem nossa avaliação detalhada de seu desafio de código e entrevista técnica. Também fornecemos o código que escreveram e acesso a uma gravação de sua entrevista técnica. [Saiba mais](https://www.g2i.co/) sobre como a G2i pode colocá-lo em parceria com o desenvolvedor perfeito para seu projeto e orçamento.

