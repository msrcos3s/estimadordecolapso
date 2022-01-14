# Estimador de Colapso de Medeiros
### autor: Marcos S. Medeiros
### Data: 12 de janeiro de 2022

Exemplo com visualização de outputs em: rpubs.com/msrcos3s/estimadordecolapso

### Sinopse

O Estimador de Colapso de Medeiros (EM) é um método em forma de ferramenta estatística inferencial de regressão linear simples desenvolvido em R proposto por Marcos S. Medeiros que prevê a data estimada do colapso do sistema de atendimento de um sistema de saúde pública com base na tendência de evolução positiva da taxa de ocupação de leitos na rede de atendimento de saúde dos últimos 10 dias, em face de qualquer contingência epidemiológica, a fim de possibilitar ações do gestor público na tomada de decisões para o enfrentamento da contingência.

O presente trabalho visa dar publicidade ao método, possibilitando seu uso por quaisquer interessados, de forma livre e gratuita, em especial pelos gestores de localidades em desenvolvimento ou com carência de pessoal na área técnica.

Embora modesta, a presente apresentação faz parte de um dos requisitos exigidos pela JHU como Capstone Project (trabalho de conclusão de curso TCC) para a obtenção do Título de Especialização em Data Science ministrado pela plataforma Coursera. Decidi, como forma de gratidão, elaborar a versão em português e, igualmente, disponibilizá-la. 

### Convenções 

Níveis de ocupação:

Med1: de 0 a =50% - faixa de demanda

Med2: de >50 a =75% - faixa de contingência 

Med3: >75% - faixa de colapso


### Utilidade da previsibilidade do colapso do sistema de saúde 

Convencionamos o marco de 75% de ocupação de leitos (Med3) como início da situação de colapso do sistema, em que o gestor público procede a medidas mais drásticas de reversão da tendência de ocupação para atender a demanda por serviços de saúde (entre elas medidas de evasão ou evitamento da causa epidemiológica provável, ampliação de leitos,  contratações emergenciais, etc). 

É de fundamental importância ao gestor saber de antemão quantos dias faltam para o colapso do sistema, a partir da tendência dos últimos 10 dias, em termos de probabilidade atual, para atingir Med3. 


### Taxa de ocupação de leitos

A taxa de ocupação de leitos, seja em enfermarias, UTI’s ou total, é uma grandeza física e matemática de fácil mensuração em qualquer região político-administrativa pré-estabelecida e é definida como sendo a quantidade de pacientes ocupando leitos hospitalares num determinado dia sobre a quantidade total de leitos existentes, e é expressa em número absoluto entre 0 e 1 ou porcentagem 0 a 100%, independentemente de qual ou quais contingências epidemiológicas são enfrentadas.

A vantagem de utilizar-se esse indicador ao longo de um período inicial de 10 dias é o fato de ser uma variável que não depende de diagnósticos conclusivos, testes clínicos ou tempo provável de permanência em internação, além de permitir uma normalização estatística de dados em eventual demora na coleta das informações ou falha nos sistemas de notificação compulsória de doenças transmissíveis. 

### Modelo de regressão linear

O pressuposto matemático para apuração do Estimador de Colapso de Medeiros é o modelo de regressão linear da taxa de ocupação de leitos dos últimos 10 dias apresentar uma inclinação positiva (β1>0).
O modelo foi desenhado para permitir que a área técnica de qualquer órgão público possa elaborar o modelo preditivo e auxiliar na tomada de decisão do gestor. 

Definiremos como data de início da medição o dia 1 da série de 10 dias. 
Seja o modelo de regressão linear simples definido como

 Y = β0 + β1*Xem 
 
onde

Y é igual à taxa de ocupação de leitos em uma data futura;

β0 é o intercepto ou a origem. É a taxa de ocupação de leitos no dia anterior ao dia 1. É o ponto que teoricamente cruza o eixo "y" no ponto zero da medição, correspondendo a uma não-medição, antes do tempo de medição. Esse ponto é calculado pelo modelo de regressão linear com base nas 10 medições.  

β1 é a tendência de ocupação, dada pela inclinação da reta do modelo de regressão linear, a ser calculada e modelada em R e

Xem é o número de dias restantes para Y.

A taxa de ocupação pode ser apresentada como número absoluto (0 a 1) ou porcentagem (0 a 100), bastando o cuidado de se manter a uniformidade da unidade estatística escolhida.

Se β1>0, então o Estimador de Medeiros (EM) será a data correspondente a X dias transcorridos a partir do dia 1 da série de observações (Xem) para Y=75% (0,75), marco convencionado para Med3. Como as informações de gestão pública são coletadas normalmente em forma de percentuais, teremos:

75 = β0 + β1*Xem

Xem=(75-β0)/β1

O Xem corresponde ao número de dias restantes para que 75% dos leitos sejam ocupados, com base na regressão linear simples das últimas 10 observações, a partir do marco zero (X0).

O modelo de regressão linear deve ser atualizado constantemente para que se tenha o dimensionamento da aceleração ou desaceleração da taxa de ocupação de leitos. Quando o EM aumenta, significa que o risco de colapso está sendo afastado; quando diminui, exige tempo de resposta mais rápido do gestor público.

**O gestor pode ajustar o nível Med3 para percentuais mais altos ou mais baixos de ocupação, bastando adaptar o modelo**. 

#### Calculando o Estimador de Colapso de Medeiros em R a partir de uma base de dados de saúde pública. 

Utilizaremos como exemplo os dados públicos do Estado de São Paulo, disponíveis na Fundação [SEADE](https://www.seade.gov.br/coronavirus) e a contingência-exemplo o aumento da ocupação de leitos em decorrência de complicações respiratórias, no período de 10 dias compreendido entre 03/01/2022 e 12/01/2022.

A escolha da base da demonstração deve-se simplesmente ao fato de tratar-se de fonte de informação confiável e disponibilizada publicamente, com quantidade de dados coletados relevantes e suficientes para o exemplo apresentado. 

Utilizaremos R e RStudio, com script em RMarkdown e exportado em Knit para disponibilização pública em RPubs e GitHub.

#### Preparando os dados para a análise

Carregando pacotes do R 
```{r}
library(ggplot2)
library(reshape2)
library(grid)
library(stringdist)
library(lubridate)
```

Download de dados
```{r}

if(!file.exists("./plano_sp_leitos_internacoes.csv")) {
        fileURL <- "https://github.com/seade-R/dados-covid-sp/blob/master/data/plano_sp_leitos_internacoes.csv"
        download.file(fileURL, "plano_sp_leitos_internacoes.csv")
}
data_raw <- read.csv("plano_sp_leitos_internacoes.csv", sep=";", dec = ",") 
dim(data_raw)
```
A base de dados do exemplo contém apenas 13.892 observações e 18 variáveis, portanto, de pequeno porte e baixa complexidade (não ocupa muito espaço, não exige performance do processador nem toma tempo de processamento em R para a modelagem).

#### Processando o modelo

O primeiro passo é realizar a mineração de dados, criando um subset com as datas e a taxa de ocupação de leitos, com o tipo de leito desejado (enfermaria, UTI, total, etc). Utilizaremos no presente desenvolvimento a taxa total do Estado de SP.
Não há um script rígido em R para se atingir os objetivos propostos, então apenas é uma demonstração dentre várias possíveis. 

Algumas redundâncias foram inseridas para melhor compreensão. 

A atribuição de nomes aos vetores criados deve fazer sentido para quem realiza o modelo, para facilidade de memorização e de reprodução dos critérios escolhidos.

```{r}
# filtrando apenas as variáveis (colunas) correspondentes a data, localidade e tipo de ocupação desejado
sub_data <- subset(data_raw,internacoes_ultimo_dia>0,select=c("datahora","nome_drs","ocupacao_leitos_ultimo_dia"))

# filtrando, a seguir, a localidade desejada e o período para o modelo
## é importante observar e reproduzir fielmente as eventuais distorções de caracteres das variáveis devido à grafia com acentuação. 
### note que foi utilizada data retroativa a 10 dias da atual (inclusive) para obter-se as 10 últimas medições diferentes de zero.

sub_data_SP <- sub_data[which(sub_data$nome_drs=='Estado de SÃ£o Paulo' & sub_data$datahora>="2022-01-03" & sub_data$datahora<="2022-01-12"), ]

SP_10 <- sub_data_SP[1:10,]

print(SP_10)
```
Temos o conjunto das últimas 10 medições de taxa de ocupação de leitos. 

O passo seguinte é transformar (ou garantir que esteja transformada) a taxa de ocupação de leitos em uma variável contínua e montar uma tabela.

```{r}
# transformando em variável contínua 

sub_data_SP$ocupacao_leitos_ultimo_dia_num <- as.numeric(sub_data_SP$ocupacao_leitos_ultimo_dia)

# montando a tabela

SP_frame <- data.frame(name=c(sub_data_SP$datahora),
                        value=c(sub_data_SP$ocupacao_leitos_ultimo_dia_num))
```

#### Montado a regressão linear simples do Estimador de Colapso de Medeiros

```{r}
# criando os vetores das 10 últimas observações
var1 <- c(seq(1,10,by=1))
var2 <- c(sub_data_SP$ocupacao_leitos_ultimo_dia_num)

# parametrizando e visualizando a regressão linear simples
EM <- lm(var2~var1)
summary(EM)
```

Os coeficientes mostram a fórmula genérica do modelo de regressão linear simples, com respectivos erros-padrão.

O (Intercept) mostra o valor de β0 e var1 mostra a inclinação da reta do modelo linear simples β1.

No exemplo dado, a fórmula de regressão linear simples, em notação decimal por ponto, é Y = 26.73 + 1.759X  SE(0.44,0.07). Considere sempre o intervalo de confiança de 95% como sendo mais duas vezes o erro padrão (SE) **e** menos duas vezes o erro padrão(SE) e o modelo estatisticamente relevante quando o p-valor for inferior a 0.05. Os valores dos coeficientes, para serem válidos, não podem incluir o zero no intervalo quando considerados os limites inferiores e superiores. Ou seja, neste exemplo, considerando o intervalo de confiança, teremos 26.73 CI95(25.85,27.61) + 1.76Xem CI95(1.62,1.9) e p<0.0001. Podemos afirmar, com segurança, com base nos dados coletados, que o modelo é estatisticamente significante. 

#### Obtendo-se o Estimador de Colapso de Medeiros (EM)

Com base na regressão linear obtida e, após validada como sendo estatisticamente significante, fazemos a substituição de Y por Med3 com base no critério epidemiológico. Na convenção aqui adotada, 75%.

75 = 26.7 + 1.7Xem

```{r}
(75-26.7)/1.7
```

O resultado de Xem=28.41 corresponde à quantidade de dias restantes até o início do colapso do sistema de saúde, caso a tendência seja mantida constante, com o mesmo número de leitos disponíveis na rede observada. Pode-se realizar novas regressões lineares diárias, com base nas últimas 10 observações, para comparar os estimadores e verificar eventual aceleração ou desaceleração do processo de colapso. 

Se β1<0, o preditor indica processo de alívio na ocupação do sistema. 

**O Estimador de Colapso de Medeiros (EM) é apresentado como uma data futura para atingir o nível Med3**.

No exemplo em tela, arredondaremos o resultado obtido para o primeiro número inteiro mais próximo, pelo critério de arredondamento. Xem=28 dias, a partir da data em X0 (dia anterior à primeira medição).

Agora, basta apresentar o EM para Med3 em forma de data futura.
EM = X1 - 1 + Xem.

Caso a data atual seja o 10º dia de observação (X10) e estiver incluída no modelo de regressão linear, podemos usar a fórmula:

EM= X10 -10 + Xem.

No presente caso, a última observação ocorreu no dia anterior. É necessário ajustar a fórmula para -1 dia.

Assim, em R, teremos
```{r}
today()-10+28-1
```
Podemos inferir, com base no modelo atual de regressão linear, que o EM é 30 de janeiro de 2022 para Med3-75. 

### Elaborando gráfico de apresentação

A primeira etapa consistirá na elaboração da informação obtida na regressão, com inserção de rótulos a critério do gestor.
```{r}
grob <- grobTree(textGrob("Y = 26.73 + 1.759X  SE(0.44,0.07)", x=0.1, y=0.95, hjust=0,
                          gp=gpar(col="red", fontsize=13, fontface="bold")))

grob2 <- grobTree(textGrob("EM para Med3-75: 30/01/2022", x=0.1,  y=0.87, hjust=0,
                          gp=gpar(col="red", fontsize=13, fontface="italic")))

grob3 <- grobTree(textGrob("modelo em R por Marcos Medeiros", x=0.1,  y=0.80, hjust=0,
                           gp=gpar(col="blue", fontsize=12, fontface="italic")))

grob4 <- grobTree(textGrob("Dados brutos em https://github.com/seade-R/dados-covid-sp", x=0.1,  y=0.73, hjust=0,
                           gp=gpar(col="orange", fontsize=12, fontface="plain")))
```
Por fim, escolhemos uma apresentação gráfica para a visualização do modelo

```{r}
ggplot(SP_frame, aes(x=name, y=value)) + 
        geom_bar(stat = "identity") + theme(axis.text.x = element_text(angle = 45, vjust = 0.5, hjust=0.5)) +
        xlab("Data") +
        ylab("% de Ocupação de Leitos") +
        ggtitle("Taxa de Ocupação de Leitos no Estado de SP")+
        scale_y_continuous(limits = c(0, 100)) + geom_smooth(method = "lm", se=FALSE, formula=y~x, color="red", aes(group=1)) +
        geom_point(color="blue")+geom_text(aes(label=value), nudge_y=3.20, color="orange")+
        annotation_custom(grob) + annotation_custom(grob2) + annotation_custom(grob3) + annotation_custom(grob4)
```
![Plot1](https://github.com/msrcos3s/estimadordecolapso/blob/main/Estimador.jpg)

### Conclusão

O Estimador de Colapso de Medeiros é um modelo de regressão linear simples que pode ser amplamente utilizado e adaptado pelo gestor público a partir de uma base de dados extensa, desde que confiável, para iniciativas e medidas de contingenciamento em saúde pública, evitando o colapso do sistema de atenção à saúde nos seus 3 níveis, servindo como indicador estatístico inferencial para a tomada de decisão. A leitura, processamento de dados e resultados são feitos em R, software estatístico gratuito e robusto, não demandando alto custo. 

Os níveis de Med1, Med2 e Med3 podem ser ajustados conforme necessidade do gestor, bem assim a quantidade de medições utilizadas para a regressão linear, conforme requeira a situação epidemiológica.

### Declaração de isenção de conflito de interesse

O autor declara que não tem nenhum conflito de interesse que possa influenciar a apresentação da presente ferramenta estatística, tais como orientação ideológica, político partidária, financeira, pessoal ou de qualquer outra natureza a não ser acadêmica.

Como bem pontua o professor de Bioestatística Roger Peng em suas aulas de especialização em Ciência de Dados, a filosofia envolvida na criação do R e o desenvolvimento de suas ferramentas é tornar o conhecimento acessível, reproduzível e útil.

Obrigado Ross. Obrigado Robert. 

### Referências

ROSNER, Bernard. _Fundamenhtals of Biostatistics_. 8a Ed. Boston: Cengage Learning, 2016.

KESTENBAUM, Brian. _Epidemiology and Biostatistics: An Introduction to Clinical Research_. Columbia: Springer, 2019.

CAFFO, Brian. _Statistical Inference for Data Science_. E-book. Disponível em:  *https://leanpub.com/LittleInferenceBook*

CAFFO, Brian. _Advanced Linear Models for Data Science_. E-book. Disponível em: *https://leanpub.com/lm*

PENG, Roger. _R Programming for Data Science_. E-book. Disponível em *https://leanpub.com/rprogramming*

PENG, Roger. _Exploratory Data Analysis with R_. E-book. Disponível em: *https://leanpub.com/exdata*

