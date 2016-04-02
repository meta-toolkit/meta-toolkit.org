---
title: Wikipedia Network Analysis
layout: page
category: demo
order: 3
---

First, you need to download and extract the following two files from [a
preprocessed version of the Wikipedia links
dataset](http://haselgrove.id.au/wikipedia.htm).

- [links-simple-sorted.zip](http://users.on.net/~henry/pagerank/links-simple-sorted.zip)

- [titles-sorted.zip](http://users.on.net/~henry/pagerank/titles-sorted.zip)

Add their paths in `config.toml` as `wiki-links` and `wiki-titles`.

Then, run the following command:
{% highlight bash %}
./wiki-page-rank config.toml
{% endhighlight %}

Note that the calculations using this dataset require about 7.5 GB of memory.

### Overall Wikipedia PageRank scores

{% highlight bash %}
1. United_States 0.00220614
2. 2007 0.00140843
3. 2008 0.00135685
4. Geographic_coordinate_system 0.00125164
5. United_Kingdom 0.00100978
6. 2006 0.000865724
7. France 0.000732849
8. Wikimedia_Commons 0.0007239
9. Wiktionary 0.000658065
10. Canada 0.000648715
11. 2005 0.000616681
12. England 0.000603392
13. Biography 0.000599893
14. Germany 0.000584397
15. United_States_postal_abbreviations 0.00055105
16. Australia 0.000528317
17. English_language 0.000518112
18. World_War_II 0.000506347
19. Japan 0.00048433
20. List_of_U._S._postal_abbreviations 0.000469503
21. Europe 0.000463587
22. India 0.000449639
23. 2004 0.000436032
24. Italy 0.00040331
25. Music_genre 0.000397291
{% endhighlight %}

### Personalized PageRank for "Computer_science"

{% highlight bash %}
1. Computer_science 5.86886e+06
2. Computer 59995
3. Programming_language 42832
4. Computational_complexity_theory 41272
5. Human-computer_interaction 38284
6. Algorithm 38085
7. Category_theory 37330
8. Data_structure 37262
9. Quantum_computer 36760
10. Cognitive_science 36476
11. Type_theory 36325
12. Computational_science 35673
13. David_Kahn 35494
14. Microarchitecture 35413
15. Mathematics 33997
16. Compiler 31904
17. 2008 31262
18. 2006 29557
19. Digital_object_identifier 28396
20. Software_engineering 28217
21. Artificial_intelligence 27915
22. Computer_programming 27734
23. Operating_system 26752
24. Association_for_Computing_Machinery 26725
25. Information_systems 26389
{% endhighlight %}

### Personalized PageRank for "Bill_Gates"

{% highlight bash %}
1. Bill_Gates 5.85817e+06
2. United_States 38556
3. 2007 34182
4. Microsoft 32890
5. 2008 32463
6. BASIC 28863
7. Seattle 28858
8. Altair_8800 28635
9. 2006 28191
10. Time_Person_of_the_Year 27929
11. Philanthropy 27777
12. United_States_Microsoft_antitrust_case 26578
13. 2005 22684
14. Operating_system 22267
15. Steve_Ballmer 21174
16. Executive_officer 20650
17. IBM 20511
18. Ray_Ozzie 20459
19. 2004 20233
20. Jeff_Raikes 20171
21. Richard_Rashid 20166
22. Craig_Mundie 19996
23. United_States_dollar 19975
24. Brian_Kevin_Turner 19854
25. Steven_Sinofsky 19838
{% endhighlight %}

### Personalized PageRank for "University_of_Illinois_at_Urbana-Champaign"

{% highlight bash %}
1. University_of_Illinois_at_Urbana-Champaign 5.87068e+06
2. United_States 38425
3. Illinois_Fighting_Illini 32101
4. UIUC_College_of_Engineering 28506
5. 2007 27630
6. UIUC_campus 26562
7. Illinois_Fighting_Illini_football 25158
8. Beckman_Institute 25084
9. Ohio_State_University 24697
10. Big_Ten_Conference 24235
11. University_of_Wisconsin-Madison 23458
12. Harvard_University 22926
13. U.S._News_&_World_Report 22882
14. Stanford_University 22729
15. Illinois 22679
16. UIUC_Main_Campus 22654
17. Illinois_Fighting_Illini_men's_basketball 22000
18. Massachusetts_Institute_of_Technology 21995
19. Yale_University 21961
20. Chief_Illiniwek 21694
21. College_and_university_rankings 21567
22. National_Center_for_Supercomputing_Applications 21542
23. 2006 21217
24. Marching_Illini 21132
25. Kenney_Gym 21125
{% endhighlight %}

### Personalized PageRank for "Pizza"

{% highlight bash %}
1. Pizza 5.86632e+06
2. United_States 51673
3. Sicilian_pizza 38670
4. Pissaladière 37739
5. Bread 37418
6. Tarte_flambée 34569
7. Italy 33079
8. Ricotta 31228
9. Olive_oil 31101
10. Farinata 30548
11. Yeast 30492
12. Chicago-style_pizza 29426
13. Wikimedia_Commons 29343
14. Pita 29341
15. Crème_fraîche 29097
16. Masonry_oven 28999
17. Calzone 28429
18. Naan 28414
19. Paratha 28092
20. Quesadilla 27985
21. Green_onion_pancake 27889
22. New_Haven-style_pizza 27786
23. Focaccia 27617
24. History_of_pizza 27600
25. Pizza_delivery 27596
{% endhighlight %}

### Personalized PageRank for "Beer"

{% highlight bash %}
1. Beer 5.86645e+06
2. Pale_lager 31153
3. Yeast 28062
4. Alcoholic_beverage 26486
5. Belgian_beer 26128
6. Brewery 24892
7. United_States 23183
8. Alcohol_by_volume 21898
9. Schnapps 21772
10. Stout 21684
11. Brewing 21470
12. 2008 20995
13. Wheat_beer 20959
14. Beer_in_the_United_States 20899
15. 2007 20413
16. Malt 20279
17. Lambic 19963
18. Beer_in_Denmark 19640
19. List_of_countries_by_beer_consumption_per_capita 19563
20. German_beer 19470
21. American_beer 19446
22. Beer_in_Africa 19417
23. Beer_in_Ireland 19279
24. Cask_ale 19191
25. Beer_in_Poland 19141
{% endhighlight %}
