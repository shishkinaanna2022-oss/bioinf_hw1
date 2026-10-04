Assembly of a bacterial genome

В рамках домашнего задания была выполнена сборка бактериального генома из paired-end и mate-pair чтений.
Из-за отсутствия доступа к серверу курса все вычисления были выполнены локально в WSL2 (Debian GNU/Linux). Исходные FASTQ-файлы предварительно скачала на локальный компьютер.

Использованные данные: 

Paired-end reads: oil_R1.fastq ; oil_R2.fastq

Mate-pair reads: oilMP_S4_L001_R1_001.fastq ; oilMP_S4_L001_R2_001.fastq


Использованные программы: seqtk, FastQC, MultiQC, Platanus, Google Colab / Python / Biopython для расчёта статистики

1. Выборка случайных чтений

Исходные FASTQ-файлы содержали большое количество чтений, поэтому перед сборкой нужно было уменьшить объём данных. С помощью программы seqtk из исходных файлов были случайным образом выбраны подмножества чтений.
Для paired-end данных было выбрано по 5 000 000 чтений из каждого файла (R1 и R2).
Для mate-pair данных было выбрано по 1 500 000 чтений из каждого файла.
Для всех четырёх файлов использовался одинаковый seed 3001. Использование одинакового seed позволяет получить воспроизводимую случайную выборку.
```
cd ~/hw1

mkdir -p subset

seqtk sample -s722 oil_R1.fastq 5000000 > subset/oil_R1.sub.fastq
seqtk sample -s722 oil_R2.fastq 5000000 > subset/oil_R2.sub.fastq

seqtk sample -s722 oilMP_S4_L001_R1_001.fastq 1500000 > subset/oilMP_R1.sub.fastq
seqtk sample -s722 oilMP_S4_L001_R2_001.fastq 1500000 > subset/oilMP_R2.sub.fastq
```
2. Контроль качества до тримминга

Для оценки качества выбранных чтений был выполнен FastQC.
Сначала была создана отдельная директория для результатов:
```
mkdir -p qc/before_trim
```
Затем для четырёх полученных FASTQ-файлов был запущен FastQC:

```
fastqc \
    subset/oil_R1.sub.fastq \
    subset/oil_R2.sub.fastq \
    subset/oilMP_R1.sub.fastq \
    subset/oilMP_R2.sub.fastq \
    -o qc/before_trim
```
    
В результате FastQC создал отдельные отчёты для каждого из четырёх FASTQ-файлов. Проверка выполнялась до тримминга, чтобы оценить исходное качество выбранных чтений.
После FastQC результаты были объединены с помощью MultiQC.
```
multiqc qc/before_trim -o qc/before_trim
```
Таким образом, MultiQC должен был собрать результаты четырёх отдельных отчётов FastQC в один общий HTML-отчёт, что позволяет удобнее сравнивать качество всех образцов.
При использовании MultiQC через терминал итоговый HTML-файл был успешно создан, однако в нём не отображались графики. Поэтому для визуального анализа результатов MultiQC была использована веб-версия MultiQC, куда были загружены результаты FastQC. Графики и статистические показатели, использованные в отчёте, были получены из веб-версии.
Таким образом, FastQC использовался для непосредственной оценки качества каждого FASTQ-файла, а MultiQC — для объединения и визуального сравнения полученных результатов.


Результаты MultiQC до тримминга:

<img width="974" height="250" alt="image" src="https://github.com/user-attachments/assets/4b746a6a-60e1-4336-aa71-195ec3f3d708" />

На рисунке представлена сводная статистика качества четырёх наборов ридов до тримминга (MultiQC).
Количество ридов соответствует ожидаемому — программа seqtk отработала корректно. Риды Mate-Pair (MP) заметно длиннее, чем Pair-End (PE). Значения GC-содержания внутри каждой пары R1/R2 практически одинаковы, что говорит о согласованности состава парных файлов.

Per Sequence GC Content

<img width="975" height="542" alt="image" src="https://github.com/user-attachments/assets/87ec5b23-5042-46e4-a593-afde07c50730" />

Все четыре образца получили статус PASS. Распределение GC имеет форму нормального распределения с пиком в районе 45–50 %. Кривые хорошо совпадают между парными ридами. Это подтверждает отсутствие систематического смещения по GC-составу и хорошее качество библиотек в этом аспекте.

Per Sequence Quality Scores

<img width="956" height="542" alt="image" src="https://github.com/user-attachments/assets/9df25277-759c-4760-9165-cf4a63267981" />

Все образцы также получили PASS. Подавляющее большинство ридов имеет высокое среднее качество (пик распределения приходится на Phred score 35–38). Доля ридов с низким средним качеством минимальна. Это показывает, что, несмотря на снижение качества к 3'-концу, общая «полезность» большинства ридов остаётся высокой.

Sequence Duplication Levels

<img width="965" height="548" alt="image" src="https://github.com/user-attachments/assets/88a5e7a0-8c8d-49b6-adc3-6c3436f130b4" />

У двух образцов (MP) наблюдается более высокая доля уникальных ридов, у двух других (PE) — повышенный уровень дубликатов (статус WARN). В целом уровень дублирования не критический.

Adapter Content

<img width="976" height="646" alt="image" src="https://github.com/user-attachments/assets/4eb73b42-b022-4a36-b7d8-9e413cd55849" />

Два образца получили FAIL, два — PASS. На графике отчётливо видно сильное накопление адаптеров к 3'-концу рида (до 70 % и выше у проблемных образцов). Это согласуется с результатами индивидуальных FastQC-отчётов пары oilMP, где доминировал Nextera Transposase Sequence.

Sequence Quality Histograms 

<img width="971" height="607" alt="image" src="https://github.com/user-attachments/assets/21891dac-5b27-46e5-8119-fc22480e7018" />

На графике видно среднее качество оснований по позициям рида. У всех образцов качество высокое в начале и середине рида (Phred ≈ 35–38), однако к 3'-концу наблюдается характерное для Illumina снижение. У двух образцов (более длинные риды MP) падение качества выражено сильнее — линии заходят в жёлтую и красную зоны. 

Для более детального анализа проблемных образцов были дополнительно рассмотрены индивидуальные отчёты FastQC пары oilMP_R1.sub и oilMP_R2.sub. Ниже приведены ключевые графики, подтверждающие выводы, полученные по MultiQC.
- Per base sequence quality — WARN у R1 и FAIL у R2. Снижение качества к 3'-концу у R2 выражено сильнее.

R1:

<img width="801" height="560" alt="image" src="https://github.com/user-attachments/assets/8ab620f4-a620-44c4-9a35-3ac76fe385c6" />


R2:

<img width="807" height="557" alt="image" src="https://github.com/user-attachments/assets/3869d580-1f54-41c7-b922-03a6e651903e" />

- Per tile sequence quality — WARN у R1 (наличие отдельных тайлов с пониженным качеством).

<img width="944" height="645" alt="image" src="https://github.com/user-attachments/assets/f2aa4d5e-9cfc-4e45-a49c-4f5fc50ca89e" />

- Adapter Content — FAIL у обоих ридов. Доминирует Nextera Transposase Sequence (до 70–75 % к концу рида), также присутствует Illumina Universal Adapter. Это указывает на то, что длина вставок в библиотеке меньше длины прочтения.

R1:

<img width="931" height="669" alt="image" src="https://github.com/user-attachments/assets/d047b16c-2857-4810-b956-6fd18303acd0" />

R2:

<img width="919" height="672" alt="image" src="https://github.com/user-attachments/assets/51416a75-7cda-42fa-8248-85bc80a3aced" />

Качество данных в целом приемлемое, однако наличие сильного адаптерного загрязнения и снижения качества на 3'-конце требует обязательного проведения тримминга.

3. Тримминг
Для paired-end чтений использовался platanus_trim:
```
cd ~/hw1

platanus_trim \
    subset/oil_R1.sub.fastq \
    subset/oil_R2.sub.fastq
```
Для mate-pair чтений использовался platanus_internal_trim:
```
platanus_internal_trim \
    subset/oilMP_R1.sub.fastq \
    subset/oilMP_R2.sub.fastq
```
4. Контроль качества после тримминга

После тримминга снова был выполнен FastQC:
```
mkdir -p qc/after_trim

fastqc \
    subset/oil_R1.sub.fastq.trimmed \
    subset/oil_R2.sub.fastq.trimmed \
    subset/oilMP_R1.sub.fastq.int_trimmed \
    subset/oilMP_R2.sub.fastq.int_trimmed \
    -o qc/after_trim
```
Затем был выполнен MultiQC:
```
multiqc qc/after_trim -o qc/after_trim
```
Результаты MultiQC после тримминга

<img width="1452" height="379" alt="image" src="https://github.com/user-attachments/assets/40174bc2-a70c-4d8f-9481-8dd1854ccb56" />

Качество данных значительно улучшилось.

Adapter Content

<img width="1485" height="968" alt="image" src="https://github.com/user-attachments/assets/f3fbb812-5cd3-46c7-9988-a24f747fafcb" />

Содержание адаптеров снизилось до минимальных значений (менее 1 %). Все образцы получили статус PASS. Это говорит об эффективном удалении адаптерных последовательностей.

Sequence Quality Histograms

<img width="1476" height="800" alt="image" src="https://github.com/user-attachments/assets/992b1195-4d13-4a2d-ae90-1bc6f2fd8c24" />

Среднее качество оснований по всей длине рида стало стабильно высоким (Phred ≈ 35–38). Падение качества к 3'-концу, наблюдавшееся до тримминга, практически исчезло. Все четыре образца получили статус PASS.

Sequence Length Distribution

<img width="1447" height="919" alt="image" src="https://github.com/user-attachments/assets/3df1eb31-96d0-4c26-a347-ee18feeb1440" />

После тримминга длина ридов уменьшилась. У образцов mate-pair средняя длина составила 105–111 bp, у образцов paired-end — около 96–98 bp. Изменение длины ожидаемо и связано с удалением адаптеров и низкокачественных участков.

Sequence Duplication Levels

<img width="1437" height="916" alt="image" src="https://github.com/user-attachments/assets/89584bd3-ff44-46f0-844c-999230019f03" />

Уровень дублирования существенно не изменился: у MP он остаётся низким (3–4 %), у PE — повышенным (30–32 %).

Тримминг успешно устранил основные проблемы исходных данных — адаптерное загрязнение и снижение качества на 3'-конце. Полученные риды имеют хорошее качество и пригодны для дальнейшего анализа.

5. Сборка контигов

Для сборки контигов из подрезанных paired-end чтений был использован platanus assemble:
```
mkdir -p assembly

platanus assemble \
    -f subset/oil_R1.sub.fastq.trimmed \
    subset/oil_R2.sub.fastq.trimmed \
    -o assembly/oil
```
6. Статистика контигов
   
Статистика была рассчитана в Google Colab с использованием Python и Biopython.

Полученные результаты:

Количество контигов	- 604

Общая длина	- 3 924 669 bp

Самый длинный контиг- 179 307 bp

N50	- 52 802 bp

Google Colab: https://colab.research.google.com/drive/1kT9EtsfpvGttNqnCA0tOc6OJ_r6eKiGY?hl=ru#scrollTo=WG_yxXuhqmeB

7. Сборка скаффолдов
Скаффолды были собраны из полученных контигов с использованием подрезанных paired-end и mate-pair чтений и сохранены в новый файл:
```
mkdir -p scaffolding

platanus scaffold \
    -c assembly/oil_contig.fa \
    -IP1 subset/oil_R1.sub.fastq.trimmed subset/oil_R2.sub.fastq.trimmed \
    -OP1 subset/oilMP_R1.sub.fastq.int_trimmed subset/oilMP_R2.sub.fastq.int_trimmed \
    -o scaffolding/from_contigs
```
8. Статистика скаффолдов
Статистика была рассчитана в Google Colab с использованием Python и Biopython.

Полученные результаты:

Количество скаффолдов - 96

Общая длина	- 3 869 763 bp

Самый длинный скаффолд - 383 582 bp

N50	- 169 908 bp

Google Colab: https://colab.research.google.com/drive/1kT9EtsfpvGttNqnCA0tOc6OJ_r6eKiGY?hl=ru#scrollTo=WG_yxXuhqmeB

9. Подсчет количества гэпов для самого длинного скаффолда

Расчеты проводились в Google Colab с использованием Python и Biopython.

Полученные результаты:

Количество гэпов - 2

Общая длина гэпов - 25 bp

Google Colab: https://colab.research.google.com/drive/1kT9EtsfpvGttNqnCA0tOc6OJ_r6eKiGY?hl=ru#scrollTo=WG_yxXuhqmeB

10. Gap closing

Для уменьшения количества гэпов был использован platanus gap_close.
В качестве входных данных использовались скаффолды и подрезанные PE и MP чтения:
```
mkdir -p gap_close

platanus gap_close \
    -c data/scaffolds.fasta \
    -IP1 subset/oil_R1.sub.fastq.trimmed subset/oil_R2.sub.fastq.trimmed \
    -OP1 subset/oilMP_R1.sub.fastq.int_trimmed subset/oilMP_R2.sub.fastq.int_trimmed \
    -o gap_close/oil \
    -t 4
```
Результат сохранились в новый файл

11.  Подсчет количества гэпов для самого длинного скаффолда после gap_closе.

Расчеты проводились в Google Colab с использованием Python и Biopython.

ID самого длинного скаффолда - scaffold15_cov234

Длина - 383 574 bp

Количество гэпов - 0

Общая длина гэпов - 0 bp

Таким образом, после выполнения gap_close в самом длинном скаффолде гэпов не осталось.

Google Colab: https://colab.research.google.com/drive/1kT9EtsfpvGttNqnCA0tOc6OJ_r6eKiGY?hl=ru#scrollTo=WG_yxXuhqmeB

12. Последовательность самого длинного скаффолда

Последовательность самого длинного скаффолда была сохранена в data/longest.fasta

Самый длинный скаффолд - scaffold15_cov234

Длина - 383 574 bp












