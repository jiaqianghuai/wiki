---
title: "PageRank算法简介及Map-Reduce实现"
layout: page
date: 2016-08-21 14:45
---

[TOC]

# 导言 #
    PageRank是对网页进行排序（重要度计算）的一种算法，是由Google创始人拉里佩奇和谢尔盖布林发明
    创造的。下文我将简单介绍PageRank算法，以及其在分布式环境下的实现方式。

## PageRank概念 ##
    PageRank算法计算每一个网页的得分值，然后根据这个值的大小对网页的重要性进行排序。该算法的思路
    是随机游走，即模拟一个悠闲的上网者，上网者首先随机选择一个网页打开，然后在这个网页上待了几分钟
    后，跳转到该网页所指向的链接，这样漫无目的得在网页上跳来跳去，PageRank就是估计这个上网者分布
    在各个网页上的概率。

## PageRank原始模型 ##
  Internet上网页之间的关系可以看成一个有向图，其中网页是结点，如果网页A有链接到网页B，则存在一
  条有向边A->B，下面是一个简单的示例：  
    1. C<-----A<----->D------>B<----->B<-----C
    2. A C D : A链向C和D
    3. B B   : B链向自己
    4. C B   : C链向B
    5. D A B : D链向A和B

    上图这个例子只有四个网页，如果当前在A页面，那么上网者将会各自有1/2的概率跳转到C和D，这里的2表
    示A有2条出链。如果一个网页有有k条出链，那么跳转任意一个出链上的概率为1/k。一般用转移矩阵表示
    上网者的跳转概率，如果用n表示网页的数目，则转移矩阵M是一个n*n的方阵，如果网页i有k个出链，那么
    对每一个出链指向网页j，有M[i][j]=1/k，而其他网页的M[i][j]=0。上面示例图对应的转移矩阵M如下：
    *     A   B   C   D
    *   A|0   0  1/2 1/2 |
    *   B|0   1   0   0  |
    *   C|0   1   0   0  |
    *   D|1/2 1/2 0   0  |

     初始时，假设上网者在每一个网页的概率都是相等的，即1/n，因而初始的概率分布是一个所有值都为1/n
     的n维列向量V0，用V0右乘转移矩阵M，得到第一步游走之后的概率分布V1=MV0。得到V1后，再用V1去右
     乘M等到V2，一直下去，直到V收敛，即Vn＝MV(n-1)。

## PageRank优化模型 ##
### 问题 ###
    1. 终止点问题
       上述游走行为是一个马尔可夫过程的实例，要满足收敛性，需要满足一个条件，即图是强联通的，即
       从任意网页可以到达其它任意网页。然而Internet上的网页不满足该条件，因而有一些网页不指向
       任何网页，这样造成的结果就是最后得到的概率分布向量所有元素几乎都为0。
    2. 陷阱问题
       有些网页不存在指向其它网页的链接，但存在指向自己的链接，比如上图的网页B。上网者跑到B网页
       后，就像跳进了陷阱，再也不能从C中出来，将导致最终概率分布值全部转移到B上来，这使得其它网
       页的概率分布值为0，从而整个网页就失去了意义。

### 解决 ###
    上面过程，我们忽略了一个问题，那就是上网者在遇到类似B网页的时候，不会停滞不前，他会在浏览器的
    地址随机输入一个地址，当然这个地址可能是原来的网页，但这里给了他一个逃离陷阱的机会。因此，对算
    法的改进，每一步，上网者都有可能不想看当前网页，不看当前网页当然也不会点击当前页面链接，而是在
    浏览器地址栏输入其它网址，而在地址栏输入跳转到其它网页的概率为1/n。假设上网者每一步查看当前网
    页的概率为a，那么其在浏览器输网址的概率为1-a，因此迭代公式转化为：
            Vn ＝ aMV(n-1) + (1-a)e, 其中e为所有值为1/n的n维单位列向量。

## PageRank计算：Map-Reduce ##
    对于互联网上的数以百亿级的网页来说，构成的转移矩阵非常大，直接按矩阵乘法的计算方法不可行，需要
    借助Map-Reduce的计算方式来解决。考虑转移矩阵是一个稀疏矩阵，我们可以用稀疏矩阵的形式表示，我
    们把web图中的每一个网页及其链出的网页作为一行，并给每个网页一个初始的pagerank值（一般为均值），这样上节的web图结构用如下方式表示：
      1. A C D
      2. B B
      3. C B
      4. D A B
      5. A a 0.25
      6. B a 0.25
      7. C a 0.25
      8. D a 0.25
### Map阶段 ###
    Map操作的每一行，对所有出链发射当前网页概率值的1/k，k是当前网页的出链数，比如对第一行输出<
    C,0.25*0.5>，<D,0.25*0.5>，其中0.25是当前网页的概率。

### Reduce阶段 ###
    Reduce操作收集id相同的网页得分值，累加并按权重计算，pi＝a*(p1+p2+...+pm)+(1-a)*1/n，
    其中m是指向网页i的网页书，n是所有网页数目。

### Python实现 ###
    //PageRank_mapper.py
    ''' mapper of pangerank algorithm'''
    import sys
    id1 = id2 = None
    heros = value = None
    count1 = 0

    for line in sys.stdin:
        data = line.strip().split('\t')
        if len(data) == 3 and data[1] == 'a':# This is the pangerank value
            count1 += 1
            if count1 >= 2:
                print '%s\t%s' % (id1,0.0)

            id1 = data[0]
            value = float(data[2])
        else:#This the link relation
            id2 = data[0]
            heros = data[1:]
        if id1 == id2 and id1:
            if (len(heros) > 0):
                v = value / len(heros)
                for hero in heros:
                    print '%s\t%s' % (hero,v)
            print '%s\t%s' % (id1,0.0)
            id1 = id2 = None
            count1 = 0

    //PageRank_reducer.py
    '''reducer of pagerank algorithm'''
    import sys
    last = None
    values = 0.0
    alpha = 0.85
    N = 59980157# Size of the web pages
    for line in sys.stdin:
        data = line.strip().split('\t')
        hero,value = data[0],float(data[1])
        if data[0] != last:
            if last:
                values = alpha * values + (1 - alpha) / N
                print '%s\ta\t%s' % (last,values)
            last = data[0]
            values = value
        else:
            values += value #accumulate the page rank value
    if last:
        values = alpha * values + (1 - alpha) / N
        print '%s\ta\t%s' % (last,values)

### C++实现 ###
    //PageRank_mapper.cc
    void mapper() {
      string line;
      string id1 = "";
      string id2 = "";
      double value = 0.0;
      int count1 = 0;
      vector<string> heros;
      while(getline(cin,line)){
          StringTrim(&line);
          vector<string> info;
          SplitStringKeepEmpty(line,"\t",&info);
          if(info.size() <= 1) continue;
          if(info.size() == 3 && info[1] == "a") { // This is the pangerank value
              count1 += 1;
              if(count1 >= 2) {
                  cout << id1 << "\t" << "0.0" << endl;
              }
              id1 = info[0];
              value = atof(info[2].c_str());
          } else { // This the link relation
              id2 = info[0];
              heros.clear();
              for (size_t i = 1; i < info.size(); i++) {
                  heros.push_back(info[i]);
              }
          }

          if (id1 == id2 && !id1.empty()) {
              if (heros.size() > 0) {
                  double v = value / heros.size();
                  for (size_t i = 0; i < heros.size(); i++){
                      cout << heros[i] << "\t" << NumberToString(v) << endl;
                  }
              }

              cout << id1 << "\t" << "0.0" << endl;
              id1 = "";
              id2 = "";
              count1 = 0;
          }
       }
    }

    //PageRank_reducer.cc
    void reducer() {
      string line;
      string last = "";
      double values = 0.0;
      double alpha = 0.85;
      int N = 59980157; //  Size of the entities
      while(getline(cin,line)){
          StringTrim(&line);
          vector<string> info;
          SplitStringKeepEmpty(line,"\t",&info);
          if(info.size() < 2) continue;
          string hero = info[0];
          StringTrim(&hero);
          string value_str = info[1];
          StringTrim(&value_str);
          double value = atof(value_str.c_str());
          if(hero != last) {
              if (!last.empty()){
                  values = alpha * values + (1 - alpha) / N;
                  cout << last << "\t" << "a" << "\t"  << NumberToString(values) << endl;
              }
              last = hero;
              values = value;
          } else {
              values += value; // accumulate the page rank value
          }
      }

      if(!last.empty()) {
          values = alpha * values + (1 - alpha) / N;
          cout << last << "\t" << "a" << "\t" << NumberToString(values) << endl;
      }
    }

### 在Linux下模仿Map-Reduce过程 ###
    for((i=1;i<=40;i++));
    do
    cat link_all | sort -n -k 1 -t \t | python PageRank_mapper.py | sort -n -k 1 -t \t | python PageRank_reducer.py > page_rank_val
    done

    ps: 用python或者c++调用hadoop的streaming，pagerank过程就能run了。有关hadoop streaming的教程可参考“董的博客>>Hadoop Streaming编程”
    http://dongxicheng.org/mapreduce/hadoop-streaming-programming/。
