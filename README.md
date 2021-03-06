<h1 align='center'> :syringe: Previsão de Admissão na UTI por COVID-19 :hospital: </h1>

#  Introdução

##  Contextualização
    
A pandemia de COVID-19 atingiu o mundo inteiro, sobrecarregando os sistemas de saúde despreparados para uma demanda tão intensa e demorada de leitos de UTI, profissionais, equipamentos de proteção individual e recursos de saúde.

Tendo como intuito de melhor preparar os sistemas de saúde e evitar colapsos, foi desenvolvido um desafio no [Kaggle](https://www.kaggle.com/S%C3%ADrio-Libanes/covid19), definidos pela necessidade de leitos de UTI acima da capacidade (presumindo-se que haja recursos humanos, EPIs e profissionais disponíveis), utilizando dados clínicos individuais - em vez de dados epidemiológicos e populacionais.


##  Objetivos
    
### Tarefa 01

Prever admissão na UTI de casos confirmados de COVID-19. Com base nos dados disponíveis, é viável prever quais pacientes precisarão de suporte em unidade de terapia intensiva? O objetivo é fornecer aos hospitais terciários e trimestrais a resposta mais precisa, para que os recursos da UTI possam ser arranjados ou a transferência do paciente possa ser agendada.
  

### Tarefa 02

Prever NÃO admissão à UTI de casos COVID-19 confirmados. Com base na subamostra de dados amplamente disponíveis, é viável prever quais pacientes precisarão de suporte de unidade de terapia intensiva? O objetivo é fornecer aos hospitais locais e temporários uma resposta boa o suficiente, para que os médicos de linha de frente possam dar alta com segurança e acompanhar remotamente esses pacientes.
  

##  Os dados
      

### Rótulo de saída (label output)
  
A Coluna ‘ICU’, ou traduzindo para o português, a UTI, deve ser considerada, como na primeira versão deste conjunto de dados, a variável alvo.

| ICU | Descrição |
|--|--|
| 0 | Não teve admissão na UTI |
| 1 | Teve admissão na UTI |

  
### Conceito de WINDOW (ou Janela)
  
A Coluna 'WINDOW' foi divida em 5 janelas de admissão na UTI.

| Window | Descrição |
|--|--|
| 0-2 | De 0 á 2 horas da admissão |
| 2-4 | De 2 a 4 horas da admissão |
| 4-6 | De 4 a 6 horas da admissão |
| 6-12 | De 6 a 12 horas da admissão|
| Above_12 | Acima de 12 horas da admissão |
  

> Cuidado para NÃO usar os dados quando a variável de alvo (‘ICU’ = 1) estiver presente, pois a ordem do evento é desconhecida (talvez o evento de destino tenha acontecido antes de os resultados serem obtidos). Eles foram mantidos lá para que possamos aumentar este conjunto de dados em outros resultados posteriormente.


### Conceito de PATIENT_VISIT_IDENTIFIER (Identificador de Paciente)

A Coluna 'PATIENT_VISIT_IDENTIFIER' é composta por 385 pacientes de identificador único, indo de 0 à 384. Cada identificador aparece 5 vezes, dentro do conjunto de dados, devido as 5 janelas de admissão.


### Conjunto de dados
  
Este conjunto de dados contém dados anônimos do Hospital Sírio-Libanês, São Paulo e Brasília. Todos os dados foram tornados anônimos de acordo com as melhores práticas e recomendações internacionais. Os dados foram limpos e escalados por coluna de acordo com o Min Max Scaler para caber entre -1 e 1.

Formato do Data Frame
| Formato | Descrição |
|--|--|
| Linhas | 1925 |
| Colunas | 231 |
  

### Dados disponíveis

1.  **Informativas** (03) 
		
		"PATIENT_VISIT_IDENTIFIER", "WINDOWS", "ICU".
2.  **Demográficas** do paciente (03)

3.  **Comorbidades** (09)

4.  **Exames laboratoriais** (36)

5.  **Sinais vitais** (06)
 
No total, são 54 recursos, expandidos quando pertinentes à média, mediana, máxima, mínima, diferença, diferença relativa (mean, median, max, min, diff e diff_rel).

6.  diff = max - min
    
7.  diff rel = diff / median
      

### Informações adicionais (dicas e truques)

**Problema**: um dos maiores desafios de trabalhar com dados de saúde é que a taxa de amostragem varia entre os diferentes tipos de medições. Por exemplo, os sinais vitais são coletados com mais frequência (geralmente de hora em hora) do que os laboratórios de sangue (geralmente diariamente).

**Dicas e truques**: É sensato supor que um paciente que não tem uma medição registrada em uma janela de tempo esteja clinicamente estável, podendo apresentar sinais vitais e exames de sangue semelhantes às janelas vizinhas. Portanto, pode-se preencher os valores ausentes usando a entrada seguinte ou anterior. Atenção aos problemas de multicolinearidade e variância zero nesses dados ao escolher seu algoritmo.

 
### Quanto mais cedo melhor!

**Problema**: A identificação precoce dos pacientes que desenvolverão um curso adverso da doença (e precisam de cuidados intensivos) é a chave para um tratamento adequado (salvar vidas) e para gerenciar leitos e recursos.

**Dicas e truques**: Considerando que um modelo preditivo usando todas as janelas de tempo provavelmente produzirá uma maior precisão, um bom modelo usando apenas o primeiro (0-2) provavelmente será mais clinicamente relevante. A criatividade é muito bem-vinda, sinta-se à vontade com a engenharia de recursos e as janelas de tempo. Atenção às medidas repetidas em indivíduos, uma vez que esses valores são (positivamente) correlacionados ao brincar com os dados.

> Para mais informações sobre os dados e a análise exploratória feita, você pode acessar os [notebooks](https://github.com/itsGab/previsao_uti_em_covid/tree/main/notebooks).
> 
> Para mais informações sobre o desafio e base de dados _raw_, você pode acessar a página do [desafio no Kaggle](https://www.kaggle.com/S%C3%ADrio-Libanes/covid19).

# Projeto

### Análise exploratória

Foi feito uma análise explatória inicial nos dados para identificarmos que tipo de dados estavam sendo apresentados.
Em seguida identificamos tipos de colunas que foram agrupadas por tipo


### Tratamento dos dados

O tratamento dos dados foi feito com:
* Remoção dos pacientes que foram admitidos na UTI na primeira janela ("0-2")
* Preparação da janela primeira janela, para a variável alvo ('UCI') igual a 1, para os pacientes que foram precisaram da UTI após a primeira janela.
* Preenchimento dados vazios, de exames laboratoriais e sinais vitais, ignorando dados obtidos após a admissão na UTI,  utilizando `pd.DataFrame.fillna(method=bfill)`
* Manutenção apenas da primeira janela, tendo em vista, a necessidade da previsão de admissão para a UTI o mais cedo possível
* Transformação dos dados "AGE_PERCENTIL" para dados categóricos, utilizando `pd.get_dummies()`.
* Remoção das colunas, de exames laboratoriais e sinais vitais, com alta correlação.
* Por fim exportação do conjunto de dados tratado.

### Modelos de ML 

Inicialmente foi usado o LazyClassifier para se ter uma base dos modelos de classificação e sua eficácia, após, foi selecionado vários modelos de classificação para se criar modelos de teste.

A métrica que foi atribuída a maior importância, foi o *recall*, métrica qual, representa o acerto dos casos positivos, ou seja, classifica com "admissão UTI" os casos que realmente são. Quanto mais próximo de um melhor (indo de 0 a 1).

Após esse verificação inicial dos modelos, foi trabalhado uma verificação com dados de treino e teste estratificados e houve um resultado completamente diferente, caindo drasticamente modelos que estavam indo bem na métrica de *recall*.

Por último, usando um *cross validation* nos modelos selecionados, o modelos apresentaram valores baixos para *recall*, em relação da importância da previsão, estando em jogo o erro do modelo, uma vida. O maior resultado de *recall* foi de 0,62.

### Conclusões

Os modelos gerados apresentam resultados muito pobres em frente a importância de suas previsões.

MODELO SE MOSTROU INEFICAZ

Para consertar isso, será necessário voltar a análise dos dados e/ou trabalhar diferentes abordagens no Machine Learning.


[O **NOTEBOOK** TESTE FINAL](https://github.com/itsGab/previsao_uti_em_covid/blob/main/notebooks/5_TESTE_FINAL_nb_para_teste_com_dados_externos.ipynb) é o designado para a testagem de dados externos ao do analisado e usado para treino.
