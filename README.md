# vk_test
# XGBoostRanker
Изначально предоставленные фичи были проверены на дубликаты - и здесь же нашлись столбцы с полностью нулевыми значениями. 
(feature_100, feature_65, feature_64, feature_72)

![Pasted image 20240429195155](https://github.com/dariamozglova/vk_test/assets/107386336/86272239-f6bb-4ffe-aa18-eda910ff44ea)

Также я нашла столбцы, которые повторяли индексы датафрейма: 
(feature_8, feature_20, feature_35)
 ![Pasted image 20240429195300](https://github.com/dariamozglova/vk_test/assets/107386336/63854bdd-35f4-4026-bbbd-16c7c0daca8c)

feature_48 из-за нормального распределения показалась мне крайне похожей на шум, после её удаления из датафрейма точность XGBoost Ranker возросла. 

 ![Pasted image 20240429205931](https://github.com/dariamozglova/vk_test/assets/107386336/ff097d10-a67a-453a-b39f-7ecdfd1e871f)

feature_95, 96, 97, 98, 99 были переведены в categorical формат. enable_categorical=True в гиперпараметрах дал небольшой прирост в точности. 

feature_54 и feature_132 также вызывали сомнения, и если убирать их независимо, они давали небольшой прирост точности, но совместное их удаление привело к снижению точности.

В процессе работы я также пыталась убрать достаточно "экстремальные" значения в метриках, однако этот шаг не дал прироста в точности, как и скейл данных - поэтому от этих шагов я отказалась.

==БЕЗ настройки гиперпараметров!==

| Особенности даты       | NDCG   |
| ---------------------- | ------ |
| baseline*              | 0.4803 |
| boxcox для данных      | 0.4799 |
| 98 перцентиль**        | 0.4790 |
| scaled(StandardScaler) | 0.4798 |

<sub> *на неизмененных данных, без удаления колонок</sub>
<sub>**заменила данные, которые оказались выше 98 перцентиля на значение 98 перцентиля</sub>

Лучшими гиперпараметрами для модели оказались:
```
tree_method="hist", lambdarank_num_pair_per_sample=150,
objective="rank:ndcg", lambdarank_pair_method="mean", enable_categorical=True

# NDCG на кроссвалидации 0.7288241916214426
```

| Особенности даты         | NDCG   |
| ------------------------ | ------ |
| С очисткой от "выбросов" | 0.7262 |
| Без очистки              | 0.7288 |

Наиболее значимые фичи после обучения:
![Pasted image 20240429202042](https://github.com/dariamozglova/vk_test/assets/107386336/ebb62780-81f5-44fd-a3b0-5a2f50421dc7)

# RankNet
Мне стало интересно, сможет ли другой алгоритм показать лучший результат. По туториалу была создана модель. (https://medium.com/swlh/ranknet-factorised-ranknet-lambdarank-explained-implementation-via-tensorflow-2-0-part-i-1e71d8923132)
![Pasted image 20240429202305](https://github.com/dariamozglova/vk_test/assets/107386336/85fd33cc-4c09-49b0-8d5e-d52ad28d5876)


early stop на 146 эпохе выдал результат: 
NDCG Train 0.6250 NDCG Validation 0.6215

![Pasted image 20240429202505](https://github.com/dariamozglova/vk_test/assets/107386336/d6ebb71f-a2c7-400d-bf4b-dc2e58960fc4)

