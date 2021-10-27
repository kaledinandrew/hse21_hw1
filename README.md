# hse21_hw1

# Домашнее задание 1 по Биоинформатике (2 год) #

## Каледин Андрей, 1 группа ##

### 1. Создадим папку для дз и ссылки на файлы ###
```
$ mkdir hw1
$ cd hw1
$ ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
$ ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
$ ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
$ ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
$ ls
oilMP_S4_L001_R1_001.fastq  oilMP_S4_L001_R2_001.fastq	oil_R1.fastq  oil_R2.fastq
```

### 2. Выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs ###
```
$ man seqtk
$ seqtk sample -s1810 oil_R1.fastq 5000000 > pe1.fastq
$ seqtk sample -s1810 oil_R2.fastq 5000000 > pe2.fastq
$ 
$ seqtk sample -s1810 oilMP_S4_L001_R1_001.fastq 1500000 > mp1.fastq
$ seqtk sample -s1810 oilMP_S4_L001_R2_001.fastq 1500000 > mp2.fastq
```

### 3. С помощью программы fastQC и multiQC оценим качество ###
```
$ mkdir fastqc
$ mkdir multiqc
$ 
$ ls pe* mp* | xargs -P 4 -tI{} fastqc -o fastqc {}
$ multiqc -o multiqc fastqc
$ ls                
fastqc	   mp2.fastq  oilMP_S4_L001_R1_001.fastq  oil_R1.fastq	pe1.fastq
mp1.fastq  multiqc    oilMP_S4_L001_R2_001.fastq  oil_R2.fastq	pe2.fastq
```

### 4. С помощью программ platanus_trim и platanus_internal_trim подрежем чтения по качеству ###
```
$ platanus_trim pe*
$ platanus_internal_trim mp*
$ ls
fastqc		              mp2.fastq.int_trimmed	      oil_R1.fastq       pe2.fastq
mp1.fastq	              multiqc			                oil_R2.fastq       pe2.fastq.trimmed
mp1.fastq.int_trimmed   oilMP_S4_L001_R1_001.fastq  pe1.fastq
mp2.fastq	              oilMP_S4_L001_R2_001.fastq  pe1.fastq.trimmed
```
```
$ rm pe1.fastq pe2.fastq mp1.fastq mp2.fastq
```

### 5. С помощью программы fastQC и multiQC оценим качество подрезанных чтений и получим по ним общую статистику ###
```
$ mkdir trimmed_fastqc
$ ls pe* mp* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
$ mkdir trimmed_multiqc
$ multiqc -o trimmed_multiqc trimmed_fastqc
```

### 6. С помощью программы “platanus assemble” собререм контиги из подрезанных чтений ###
```
$ platanus assemble -o Poil -t 1 -m 16 -f pe1.fastq.trimmed pe2.fastq.trimmed 2>assemble.log
$ ls
assemble.log	          multiqc			                oil_R2.fastq       Poil_contigBubble.fa
fastqc		              oilMP_S4_L001_R1_001.fastq  pe1.fastq.trimmed  Poil_contig.fa
mp1.fastq.int_trimmed   oilMP_S4_L001_R2_001.fastq  pe2.fastq.trimmed  trimmed_fastqc
mp2.fastq.int_trimmed   oil_R1.fastq		            Poil_32merFrq.tsv  trimmed_multiqc
```

### 7. С помощью программы “platanus scaffold” собререм скаффолды из контигов, а также из подрезанных чтений ###
```
$ platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 pe1.fastq.trimmed pe2.fastq.trimmed -OP2 mp1.fastq.int_trimmed mp2.fastq.int_trimmed 2>scaffold.log
$ ls
assemble.log		            oilMP_S4_L001_R2_001.fastq	Poil_contigBubble.fa	    Poil_scaffold.fa
fastqc			                oil_R1.fastq		            Poil_contig.fa		        scaffold.log
mp1.fastq.int_trimmed	      oil_R2.fastq		            Poil_lib1_insFreq.tsv	    trimmed_fastqc
mp2.fastq.int_trimmed	      pe1.fastq.trimmed		        Poil_lib2_insFreq.tsv	    trimmed_multiqc
multiqc			                pe2.fastq.trimmed		        Poil_scaffoldBubble.fa
oilMP_S4_L001_R1_001.fastq  Poil_32merFrq.tsv		        Poil_scaffoldComponent.tsv
```

### 8. С помощью программы “platanus gap_close” уменьшим кол-во гэпов с помощью подрезанных чтений ###
```
$ platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 pe1.fastq.trimmed pe2.fastq.trimmed -OP2 mp1.fastq.int_trimmed mp2.fastq.int_trimmed 2> gapclose.log
$ ls
assemble.log	          oilMP_S4_L001_R1_001.fastq  Poil_32merFrq.tsv	      Poil_scaffoldBubble.fa
fastqc		              oilMP_S4_L001_R2_001.fastq  Poil_contigBubble.fa    Poil_scaffoldComponent.tsv
gapclose.log	          oil_R1.fastq		            Poil_contig.fa	        Poil_scaffold.fa
mp1.fastq.int_trimmed   oil_R2.fastq		            Poil_gapClosed.fa	      scaffold.log
mp2.fastq.int_trimmed   pe1.fastq.trimmed	          Poil_lib1_insFreq.tsv   trimmed_fastqc
multiqc		              pe2.fastq.trimmed	          Poil_lib2_insFreq.tsv   trimmed_multiqc
```

### 9. Удалим исходные .fastq файлы, полученные с помощью программы seqtk ###
```
$ rm pe1.fastq.trimmed pe2.fastq.trimmed mp1.fastq.int_trimmed mp2.fastq.int_trimmed
```

## Colab с анализом файлов ##
https://colab.research.google.com/drive/1PNGioxrFurz_mejnZ8NF3q8NmGZupiUP?usp=sharing
