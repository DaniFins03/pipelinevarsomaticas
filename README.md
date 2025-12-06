# pipelinevarsomaticas
Filtragem de variantes somáticas do VCF até CGI

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

**5.Log completo do JoID**

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

**6.Download dos resultados**

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









