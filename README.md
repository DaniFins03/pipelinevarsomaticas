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
