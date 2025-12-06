## Pipeline de variantes de câncer somáticas ##
Nesta aula, iremos aprender o passo a passo de filtragem de variantes de câncer somáticas do VCF até o Cancer Genome interpreter (CGI), na busca de distinguir variantes drivers ou passengers de câncer do seu VCF! Neste exemplo específico, iremos processar a amostra **WP048**. Ao final, teremos o link do Google Colab, que demonstra o mesmo pipeline aplicado para as amostras **WP017, WP019, WP058 e WP068**. E também, uma tabela geral, que reúne informações das 5 amostras analisadas.

**1. Clonar o git imabrasil-hg38**

```bash
!git clone https://github.com/renatopuga/lmabrasil-hg38.git
```

output esperado:
```
Cloning into 'lmabrasil-hg38'...
remote: Enumerating objects: 226, done.
remote: Counting objects: 100% (168/168), done.
remote: Compressing objects: 100% (108/108), done.
remote: Total 226 (delta 90), reused 114 (delta 56), pack-reused 58 (from 1)
Receiving objects: 100% (226/226), 8.63 MiB | 29.08 MiB/s, done.
Resolving deltas: 100% (106/106), done.
```



**2. Agora, vá até o github lmabrasil-hg38 na sessão usando CGI via API Rest no Google Colab**

```bash
%%bash
# Vai cortar pelas colunas de 1 a 4 e criar um novo arquivo chamado df_WP048-cgi.txt
# 1: CHROM (converte para CHR) - formato que o CGI gosta
# 2: POS
# 3: REF 
# 4: ALT
cut -f1-4 /content/lmabrasil-hg38/vep_output/liftOver_WP048_hg19ToHg38.vep.filter.tsv | sed -e "s/CHROM/CHR/g"  > df_WP048-cgi.txt

#Listar as 10 primeiras linhas (se tiver 10)
head df_WP048-cgi.txt
```

Output esperado: 
```
CHR	POS	REF	ALT
chr1	114716123	C	T
chr9	5073770	G	T
```



**3. Enviar job para Cancer Genome Interpreter (CGI) API**
>Fonte: https://www.cancergenomeinterpreter.org/rest_api

Após filtrar apenas as colunas de interesse (CHR, POS, REF, ALT), agora podemos enviar vir REST-API as variantes somáticas da amostra WP048.

Nota: Altere a variável {SEU_TOKEN} para o Token do CGI criado para sua conta.

```python
import requests
headers = {'Authorization': 'danifinsbiomed@gmail.com {SEU_TOKEN}'}
payload = {'cancer_type': 'HEMATO', 'title': 'Somatic MF WP048', 'reference': 'hg38'}
r = requests.post('https://www.cancergenomeinterpreter.org/api/v1',
                headers=headers,
                files={
                        'mutations': open('/content/df_WP048-cgi.txt', 'rb')
                        },
                data=payload)
r.json()
```

Output esperado: 
```
c22ecf2af788d65ec257
```



**4. Status do JobID**

Verifique o `status` do seu JobID para identificar se a análise terminou ou se houve algum erro.

Status possíveis são: Running, Error ou Done.

```python
import requests
job_id ="c22ecf2af788d65ec257"

headers = {'Authorization': 'danifinsbiomed@gmail.com {SEU_TOKEN}'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers)
r.json()
```

Output esperado: 
```
{'status': 'Done',
 'metadata': {'id': 'c22ecf2af788d65ec257',
  'user': 'danifinsbiomed@gmail.com',
  'title': 'Somatic MF WP048',
  'cancertype': 'HEMATO',
  'reference': 'hg38',
  'dataset': 'input.tsv',
  'date': '2025-12-06 14:05:09'}}
```



**5. Log completo do JoID**

Aqui, podemos verificar o status em cada etapa do CGI.

``` python
import requests
job_id ="c22ecf2af788d65ec257"

headers = {'Authorization': 'danifinsbiomed@gmail.com {SEU_TOKEN}'}
payload={'action':'logs'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers, params=payload)
r.json()
```

Output esperado: 

```
{'status': 'Done',
 'logs': ['# cgi analyze input.tsv -c HEMATO -g hg38',
  '2025-12-06 15:05:13,869 INFO     Parsing input01.tsv\n',
  '2025-12-06 15:05:17,384 INFO     Running VEP\n',
  '2025-12-06 15:05:18,781 INFO     Check cancer genes and consensus roles\n',
  '2025-12-06 15:05:18,859 INFO     Annotate BoostDM mutations\n',
  '2025-12-06 15:05:19,044 INFO     Annotate OncodriveMUT mutations\n',
  '2025-12-06 15:05:21,677 INFO     Annotate validated oncogenic mutations\n',
  '2025-12-06 15:05:21,830 INFO     Check oncogenic classification\n',
  '2025-12-06 15:05:21,891 INFO     Matching biomarkers\n',
  '2025-12-06 15:05:21,979 INFO     Prescription finished\n',
  '2025-12-06 15:05:21,990 INFO     Aggregate metrics\n',
  '2025-12-06 15:05:29,952 INFO     Compress output files\n',
  '2025-12-06 15:05:30,008 INFO     Analysis done\n']}
```



**6. Download dos resultados**

Total de 4 arquivos de resultado: 
>Definicar cada um deles com base na documentação do CGI (TODOS - ver no site)
- alterations.tsv: ...
- biomarkers.tsv: ...
- input01.tsv: ...
- summary.txt: ...

```
%%bash
#Criar diretório com ID da amostra dentro de results
mkdir -p results/WP048
```

```
Output esperado:
import requests
job_id ="c22ecf2af788d65ec257"

headers = {'Authorization': 'danifinsbiomed@gmail.com 3f6ba960478b9fc33a5b'}
payload={'action':'download'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers, params=payload)
with open('/content/results/WP048/WP048-cgi.zip', 'wb') as fd:
    fd.write(r._content)
```



 **7. Descompactar o zip com os resultados**

 ```
%%bash 
unzip /content/results/WP048/WP048-cgi.zip -d /content/results/WP048/
```

Output esperado: 

```
Archive:  /content/results/WP048/WP048-cgi.zip
  inflating: /content/results/WP048/alterations.tsv  
  inflating: /content/results/WP048/biomarkers.tsv  
  inflating: /content/results/WP048/input01.tsv  
  inflating: /content/results/WP048/summary.txt
```



**8. Visualizar a tabela** `alterations.tsv`

Instalar lib pandas `pip install pandas `
  - O pandas é uma instalação que permite algumas funções, como a criação visual de tabela. Vamos baixar ele exatamente para isso, pois queremos visualizar o resultado em forma de tabela.

```
%%bash 
pip install pandas
```

Output esperado: 

```
Requirement already satisfied: pandas in /usr/local/lib/python3.12/dist-packages (2.2.2)
Requirement already satisfied: numpy>=1.26.0 in /usr/local/lib/python3.12/dist-packages (from pandas) (2.0.2)
Requirement already satisfied: python-dateutil>=2.8.2 in /usr/local/lib/python3.12/dist-packages (from pandas) (2.9.0.post0)
Requirement already satisfied: pytz>=2020.1 in /usr/local/lib/python3.12/dist-packages (from pandas) (2025.2)
Requirement already satisfied: tzdata>=2022.7 in /usr/local/lib/python3.12/dist-packages (from pandas) (2025.2)
Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.12/dist-packages (from python-dateutil>=2.8.2->pandas) (1.17.0)

[ ]
```

Imprimir tabela com alterações: 

```
pd.read_csv('/content/alterations.tsv',sep='\t',index_col=False, engine= 'python')

```
Output esperado:

|index|Input ID|CHROMOSOME|POSITION|REF|ALT|CHR|POS|ALT\_TYPE|STRAND|CGI-Sample ID|CGI-Gene|CGI-Protein Change|CGI-Oncogenic Summary|CGI-Oncogenic Prediction|CGI-External oncogenic annotation|CGI-Mutation|CGI-Consequence|CGI-Transcript|CGI-STRAND|CGI-Type|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|input01\_1|1|114716123|C|T|chr1|114716123|snp|+|input01|NRAS|G13D|oncogenic \(predicted and annotated\)|driver \(boostDM: non-tissue-specific model\)|cgi,oncokb,clinvar:13901|chr1:114716123 C\>T|missense\_variant|ENST00000369535|+|SNV|
|1|input01\_2|9|5073770|G|T|chr9|5073770|snp|+|input01|JAK2|V617F|oncogenic \(annotated\)|passenger \(oncodriveMUT\)|cgi,oncokb,clinvar:14662|chr9:5073770 G\>T|missense\_variant|ENST00000381652|+|SNV|

Nota: Para gerar a tabela no girhub, puxada do collab, é necessário colocar no modo de visualização de tabela selecionando o unico no canto superior direito do output, para uma tabela em formato melhor, selecionar o ícone de "cópia" e selecionar a opção "markdown". Após isso, colar no github (para uma visualização mais homegênea).

Nota 2: Outra opção, é utilizar o site https://www.tablesgenerator.com/markdown_tables (copiar a tabela no site).



**Código de processamento das amostras WP017, WP019, WP058 e WP068**

Link para o google Colab : https://colab.research.google.com/drive/1NgdeDoazeNgxKI9XA7X3U1KLE14ycaY3?usp=sharing

Obs: Vale destacar que, em caso de o pandas já ter sido instalado no processamento de uma das amostras, não há necessidade de instalá-lo de novo, contanto que não saia da página e o servidor não desconecte. 


**Tabela Geral das variantes somáticas de câncer presentes nas amostras**













