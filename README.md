# BIOCAD_A2aAR

**Кейс:** Изучить фокусированную библиотеку ингибиторов A2aAR. Предложить способ поиска в ней потенциально активных молекул. Выбрать несколько возможных кандидатов и объяснить свой выбор.

[Ссылка для скачивания данных](https://drive.google.com/drive/folders/1qlieTlBvTo_p6yKORecSkwkn4s9s-65T)

## Препроцессинг

Файл: [non-specific_filter.ipynb](https://github.com/alyferryhalo/BIOCAD_A2aAR/blob/main/non-specific_filter.ipynb)

Прежде всего попробуем провести виртуальный скрининг. Библиотека, безусловно, уже является фокусированной, но соединений в ней по-прежнему достаточно много: **20323** штук. Надо попробовать отобрать по крайней мере “хорошие” соединения простейшим отбором.

Для начала проведём неспецифическое фильтрование (препроцессинг), чтобы выбрать соединения, обладающие наиболее приемлемыми свойствами. Для этого отфильтруем соединения по их физико-химическим свойствам (правила Липински, правила “lead-likeness” и “drug-likeness”), отфильтруем соединения по токсофорам. Затем планировалось применить подструктурные фильтры (PAINS, например), однако с ними возникли некоторые проблемы. К тому же, есть мнение, что процент ложных срабатываний у них бывает слишком высок.

По правилу Липински биодоступность оптимальна, если:
* H-bond donors < 5
* MW < 500
* LogP < 5
* H-bond acceptors < 10

По критериям “lead-likeness”:
* Rotatable bonds < 10
* Rings > 0
* Chiral centers < 3

Для исключения соединений, содержащих наиболее распространённые токсофоры, я воспользовалась [scopy.ScoTox](https://scopy.iamkotori.com/modules/scopy.ScoTox.html).

По итогам препроцессинга (Липински, частично lead-likeness, токсофоры) осталось **2138** молекул, найти которые можно в [accepted_smiles.txt](https://github.com/alyferryhalo/BIOCAD_A2aAR/blob/main/accepted_smiles.txt). Получается, удалось сократить датасет примерно на 90%.

## Ранжирование с использованием 2D-структур 

Сходство с активными соединениями (индекс Танимото для сравнения фингерпринтов) — виртуальный скрининг на основе молекулярного подобия (SBVS). Представляется логичным, что если некая молекула похожа на молекулу с уже установленной активностью, то вероятность того, что исследуемая молекула будет обладать требуемой активностью, выше. Поэтому для данного фрагмента исследования я решила взять несколько формул известных ингибиторов A2aAR и посчитать инжекс Танимото для оставшихся в датасете молекул, чтобы выделить среди них наиболее похожие.

Соединения для сравнения были взяты [отсюда.](https://en.wikipedia.org/wiki/Adenosine_A2A_receptor#Antagonists)

Был рассчитан индекс Танимото для всех отобранных ранее молекул. Файл можно посмотреть [вот тут.](https://github.com/alyferryhalo/BIOCAD_A2aAR/blob/main/tanimoto_similarity.ipynb) Было решено отобрать все молекулы, у которых индекс Танимото для той или иной референсной молекулы был больше 0.25. 

В результате получился [список](https://github.com/alyferryhalo/BIOCAD_A2aAR/blob/main/tanimoto_smiles.txt) из **47** молекул. Это немного меньше (сильно меньше), чем я ожидала, но исследование продолжается.

## Ранжирование с использованием 3D-структур

Для дальнейших исследований возьмём структуру [4EIY](https://www.rcsb.org/structure/4eiy) в формате .pdb

Полученные на предыдущем шаге SMILES конвертируем в формат **.pdbqt**, подходящий для докинга в Autodock, при помощи [open babel.](http://www.cheminfo.org/Chemistry/Cheminformatics/FormatConverter/index.html) Получившиеся файлы находится [здесь.](https://github.com/alyferryhalo/BIOCAD_A2aAR/tree/main/molecules)

Прежде всего удалим лишние молекулы, которые "привязались" к этой структуре. Затем необходимо приготовить саму молекулу белка и проверить, нет ли отсутствующих атомов:

![image](https://user-images.githubusercontent.com/61160686/164845065-f5f8386a-37c3-4305-8f66-e08c27c22030.png)

Теперь добавим полярные водороды:

![image](https://user-images.githubusercontent.com/61160686/164845084-3b622979-d75d-4b4c-be89-c1229b2fcfd2.png)

И Kollman Charges:

![image](https://user-images.githubusercontent.com/61160686/164845098-a0ab7346-7ee6-43fc-9a6e-a081166ed561.png)

Теперь проверим, равномерно ли распределены заряды. Оказывается, что нет:

![image](https://user-images.githubusercontent.com/61160686/164845176-abae4c76-c352-4fae-88f9-5e8e7a9b72fc.png)

Распределим заряды равномерно:

![image](https://user-images.githubusercontent.com/61160686/164845198-5f54d1ae-7cce-4a71-9d61-a8de33927b40.png)

И проверим снова. Теперь всё в порядке:

![image](https://user-images.githubusercontent.com/61160686/164845163-53eeb4ee-89f8-4a05-9185-c0a7f61f96d2.png)

Наконец, можно добавлять лиганды. Однако, как обычно, что-то пошло не по плану, и ни один лиганд не был открыт, из-за чего вся дальнейшая работа полностью остановилась:

![image](https://user-images.githubusercontent.com/61160686/164944547-c1f1537d-1601-4b10-b49c-d0c76771c710.png)

Файл с "чистой" структурой (вместе с сайтами связывания, которые были сочтены AutoDock наиболее интересными) был сохранён как [4eiy_clean.pdb](https://github.com/alyferryhalo/BIOCAD_A2aAR/blob/main/4eiy_clean.pdb).

Ввиду проблем с AutoDock было принято решение использовать Schrodinger Maestro.

## Schrodinger Maestro

### Ligand Preparation

Импортируем структуры из файла [tanimoto_smiles.txt](https://github.com/alyferryhalo/BIOCAD_A2aAR/blob/main/tanimoto_smiles.txt) и обрабатываем при помощи LigPrep:

<img width="445" alt="Screenshot 2022-08-24 at 19 19 16" src="https://user-images.githubusercontent.com/61160686/186485048-fea38d75-e08a-42a3-a13e-4e78e3e4c4e5.png">

### Protein Preparation

При помощи Protein Wizard Preparation оптимизируем структуру 4EIY:

<img width="545" alt="Screenshot 2022-08-24 at 19 22 26" src="https://user-images.githubusercontent.com/61160686/186485644-850cd6e2-d154-44c9-bd16-34a33b07968d.png">

<img width="545" alt="Screenshot 2022-08-24 at 19 22 26" src="https://user-images.githubusercontent.com/61160686/186485681-5f7cd7f5-c569-44f6-9b9b-0d1543749762.png">

Посмотрим заодно на лиганд:

<img width="389" alt="Screenshot 2022-08-24 at 19 34 05" src="https://user-images.githubusercontent.com/61160686/186485750-2b28363f-b5e1-4d91-a890-f81f9919b094.png">

И его позу:

![standard_ligand_interaction_map](https://user-images.githubusercontent.com/61160686/186485771-c7684e04-b8b9-4e42-8b4a-58f6cec80754.png)

### Генерация решётки

<img width="702" alt="Screenshot 2022-08-24 at 19 40 16" src="https://user-images.githubusercontent.com/61160686/186485922-fc737d48-5bb9-4abe-8272-9413c4612dd5.png">

<img width="702" alt="Screenshot 2022-08-24 at 19 42 28" src="https://user-images.githubusercontent.com/61160686/186485940-8945fbef-98de-483a-b550-69d41065367c.png">

### Параметры докинга

<img width="702" alt="Screenshot 2022-08-24 at 19 57 48" src="https://user-images.githubusercontent.com/61160686/186487966-80d14f35-3424-42ec-bfc9-1f2a5b82d024.png">

<img width="702" alt="Screenshot 2022-08-24 at 19 57 52" src="https://user-images.githubusercontent.com/61160686/186487778-861ac3aa-cec2-4035-a615-deedb0a66e20.png">

Полученные результаты находятся в файле [mols_docking.csv](https://github.com/alyferryhalo/BIOCAD_A2aAR/blob/main/mols_docking.csv).

## Карты взаимодействия кандидатов

### COc1ccc(-c2nn(CCCC(=O)NCCc3ccc(Cl)cc3)c(=O)c3noc(C)c23)cc1

![COc1ccc(-c2nn(CCCC(=O)NCCc3ccc(Cl)cc3)c(=O)c3noc(C)c23)cc1](https://user-images.githubusercontent.com/61160686/186491357-dcf72495-0092-4203-92b4-e15cbf2d2856.png)

### Cn1ncc2c(NCc3ccccc3)nc(N3CCN(CCO)CC3)nc21

![Cn1ncc2c(NCc3ccccc3)nc(N3CCN(CCO)CC3)nc21](https://user-images.githubusercontent.com/61160686/186491382-9e22fede-fe5a-407e-9956-2cf07d930db2.png)
