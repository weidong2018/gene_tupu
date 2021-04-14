# gene_tupu
一. 生物知识图谱介绍
欢迎来到生物知识图谱的查询页面，该知识图谱由复旦大学知识工场实验室构建，包含5类节点（共3.4亿个实体），12类关系（共1.23亿条边）：

节点信息
GO（基因本体 gene ontology）
本质上就是一个生物学相关名词的词表，大致有生物学过程、细胞组分以及分子功能等三类，可由其连接的边进行判断。

Gene（基因）
每个节点对应的是特定基因，有两个属性——对应的基因ID（id：xxx）以及所属物种（species：xxx）。

Protein（蛋白质）
每个节点对应的是特定蛋白质，有三个属性——对应的蛋白质id（id：xxx）、对应的功能（function：xxx）以及人工标注信息（comment：xxx）。

Sequence（序列）
每个节点对应的某个基因转录出来的某个序列（对应be_transcribed_into这个边），可能某个蛋白质的组成部分（对应belong_to这个边），目前只有一个属性——对应的序列ID（id：xxx）。

Species（物种）
目前数据库里面只有植物，目前只有一个属性——对应的物种名（name：xxx）。

边信息
目前库里面有12类边，大致可以分成以下几类：

Species拥有某个Gene——have。
Gene转录为某个Sequence——be_transcribed_into。
GO内部的关系：包括以下几类边——is_a, negatively_regulates, positively_regulates, part_of, regulates。
利用GO对Gene进行标注：包括以下几类边——molecular_function, biological_process, cellular_component, eco。
Sequence序列所属蛋白质——belong_to。
二. 知识图谱查询功能介绍
针对上述的描述，主要实现的是GO、Gene、Protein、Sequence、Species这五类节点的查询， 对前四类节点的查询反馈的信息分为两个部分——一个就是节点本身的属性，另一个就是该节点的邻居信息（包括连接其的边）。 Species的查询返回该Species拥有的Gene的列表。 具体来说每类节点展示信息如下：
GO（基因本体 gene ontology）
两类信息——
与该生物概念相关的基因（对应molecular_function、biological_process、cellular_component、eco等边，不同的边代表不同的关系）；
以及该实体在生物概念中所处的地位（对应is_a、negatively_regulates、positively_regulates、part_of、regulates等边）。
例：GO:0000019
例：GO:0045771
例：GO:1904098

Gene（基因）
两类信息——
基因ID、物种（本身属性）；
参与生物过程、所处细胞结构、具备分子功能、转录产物（由其相邻的节点可以获得， 分别对应molecular_function、biological_process、cellular_component、be_transcribed_into等边）。
例：AET1Gv20674500
例：AET5Gv20917300
例：AET2Gv20623500

Protein（蛋白质）
两类信息——
包含序列（由belong_to逆向得到）；
具备相同序列的蛋白质（由belong_to逆向找到对应Sequence，将该Sequence属于的所有蛋白质列出来）。
例：A0A0N1IVW0
例：A0A0N1IVW1
例：A0A0E4BJQ2

Sequence（序列）
两类信息——
原始基因（由be_transcribed_into逆向得到）；
蛋白质产物（即该序列对应的肽链所属的蛋白质，由belong_to得到）。
例：UPI000267421C
例：UPI0000DB4D33
例：UPI000005F8C8

Species（物种）
一类信息——
该Species拥有的Gene的列表（由have得到）。
例：Aegilops tauschii
例：Gossypium raimondii
例：Medicago truncatula

三. 知识图谱问答功能介绍
用户在问答框中输入问题，问答功能通过问题进行实体识别，并将问题与模版的问题（模版中针对每类关系有4、5种）进行相似度匹配， 从而进行neo4j查询来回答相关问题。针对用户可能对实体Species拼错、漏写字母的情况， 问答系统会模糊查询出用户想要提问的实体，具有一定容错性。

输入格式
问题中输入的基因本体GO的序列以如下形式：GO 0043234
问题中每个单词、符号之间需有空格：
例：Is this gene ontology GO 0060255 a subcategory of the gene ontology GO 0051052 ?
对不同关系类型提问的问题示例
这部分根据不同关系类型列举问题示例。可输入问答框后点击Ask查看回答。
关系类型汇总：eco、biological_process、cellular_component、molecular_function、 be_transcribed_into、belong_to、have、is_a、negatively_regulates、 positively_regulates、part_of、regulates

1 eco (证据与结论本体evidence & conclusion ontology)：
关系说明：Gene-->GO
例：What genes is an evidence ontology in Sorghum bicolor ?
例：How many genes are involved in the evidence and conclusion ontology of gene ontology GO 0043234 ?
例：Does the gene Bra023371 participate in the evidence and conclusion ontology of gene ontology GO 0043234 ?

2 biological_process(参与生物进程)：
关系说明：Gene-->GO
例：What genes do biological process includes in Vitis vinifera ?
例：How many genes are involved in this biological process of gene ontology GO 0006281 ?
例：Does this gene AET5Gv20554800 participate in this biological process of gene ontology GO 0055114 ?

3 cellular_component(细胞组分):
关系说明：Gene-->GO
例：what genes do cellular component in Vitis vinifera ?
例：How many genes are contained in the cellular components of gene ontology GO 0009941 ?
例：Does the cell component of gene ontology GO 0009941 contain this gene AET4Gv20696400 ?

4 molecular_function(分子功能):
关系说明：Gene-->GO
例：what genes do molecular function includes in Vitis vinifera?
例：How many genes perform molecular functions on this gene ontology GO 0005515 ?
例：Does this gene AET3Gv21167000 perform molecular functions on this gene ontology GO 0005515 ?

5 be_transcribed_into(转录):
关系说明：Gene--> Protein/Sequence
例：Gene Os01g0740400 is transcribed into what sequence ?
例：Gene id BGIOSGA035786 is transcribed into what protein ?
6 belong_to(属于):
关系说明：Sequence--> Protein
例：Sequence UPI0000000444 belongs to which protein ?
7 have(具有):
关系说明：Species--> Gene
例：What genes do Brassica napus have ?
例：How many genes exist in this species Brassica napus ?
例：Does this gene AET4Gv20649600 exist in this species Aegilops tauschii ?

8 is_a(子类):
关系说明：GO--> GO
例：what is the GO that has relationship " is a " with GO 0060255 ?
例：How many subcategories are there in this gene ontology GO 0051052 ?
例：Is this gene ontology GO 0060255 a subcategory of the gene ontology GO 0051052 ?

9 negatively_regulates(负向调节):
关系说明：GO--> GO
例：Which gene ontology GO does the gene ontology GO 0006351 negative regulate ?
例：How many gene ontologies GO does the gene ontology GO 0006351 negative regulate ?
例：Does the gene ontology GO 0006351 negative regulate the gene ontology GO 0045892 ?

10 positively_regulates(积极调节):
关系说明：GO--> GO
例：what is the GO that has relationship " positively regulates " with GO 0044700 ?
例：How many gene ontologies GO does this gene ontology GO 0044700 positive regulate ?
例：Does the gene ontology GO 0044700 positive regulate the gene ontology GO 0023056 ?

11 part_of(部分)：
关系说明：GO--> GO
例：What part of this gene ontology GO 0006886 ?
例：How many parts of this gene ontology GO 0051318 are there?
例：This gene ontology GO 0042254 is part of which gene ontology GO ?

12 regulates(调节)：
关系说明：GO--> GO
例：Which gene ontology GO has a regulatory effect ?
例：How many gene ontologies GO does this gene ontology GO 0006366 regulate?
例：Does the gene ontology GO 0006351 regulate that gene ontology GO 0006355 ?

13 regulates(调节)：
关系说明：GO--> GO
例：Which gene ontology GO has a regulatory effect ?
例：How many gene ontologies GO does this gene ontology GO 0006366 regulate?
例：Does the gene ontology GO 0006351 regulate that gene ontology GO 0006355 ?

14 其他：
例：What species are in the biological gene graph ?
例：What genes are in the biological gene graph ?
例：What gene ontologies GO are in the biological gene graph ?
例：What sequences are in the biological gene graph ?
例：What proteins are in the biological gene graph ?
