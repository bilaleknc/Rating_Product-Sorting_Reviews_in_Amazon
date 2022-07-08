######################################################################
# Rating Product & Sorting Reviews in Amazon
#######################################################################




# İŞ PROBLEMİ
"""
 E-ticaretteki en önemli problemlerden bir tanesi ürünlere satış sonrası verilen puanların doğru şekilde hesaplanmasıdır.
 Bu problemin çözümü e-ticaret sitesi için daha fazla müşteri memnuniyeti sağlamak, satıcılar için ürünün öne çıkması ve
 satın alanlar için sorunsuz bir alışveriş deneyimi demektir. Bir diğer problem ise ürünlere verilen yorumların doğru
 bir şekilde sıralanması olarak karşımıza çıkmaktadır. Yanıltıcı yorumların öne çıkması ürünün satışını doğrudan
 etkileyeceğinden dolayı hem maddi kayıp hem de müşteri kaybına neden olacaktır. Bu 2 temel problemin çözümünde
 e-ticaret sitesi ve satıcılar satışlarını arttırırken müşteriler ise satın alma yolculuğunu sorunsuz olarak tamamlayacaktır.
"""



# Veri Seti Hikayesi




"""
Amazon ürün verilerini içeren bu veri seti ürün kategorileri ile çeşitli metadataları içermektedir. Elektronik 
kategorisindeki en fazla yorum alan ürünün kullanıcı puanları ve yorumları vardır.
"""

# reviewerID	    Kullanıcı ID’si
# asin	            Ürün ID’si
# reviewerName	    Kullanıcı Adı
# helpful	        Faydalı değerlendirme derecesi
# reviewText	    Değerlendirme
# overall	        Ürün rating’i
# summary	        Değerlendirme özeti
# unixReviewTime	Değerlendirme zamanı
# reviewTime	    Değerlendirme zamanı Raw
# day_diff	        Değerlendirmeden itibaren geçen gün sayısı
# helpful_yes	    Değerlendirmenin faydalı bulunma sayısı
# total_vote	    Değerlendirmeye verilen oy sayısı

import pandas as pd
import math
import datetime as dt
import scipy.stats as st
df = pd.read_csv("C:/Users/Victus/Desktop/VBO/projeler/Rating_Sorting_Reviews_Amazon/amazon_review.csv")




##################################################################################################
# 1. Average Rating’i güncel yorumlara göre hesaplayıp ve var olan average rating ile kıyaslayalım.
##################################################################################################

"""
Paylaşılan veri setinde kullanıcılar bir ürüne puanlar vermiş ve yorumlar yapmıştır. Amacımız verilen 
puanları tarihe göre ağırlıklandırarak değerlendirmek. İlk ortalama puan ile elde edilecek tarihe göre ağırlıklı
puanın karşılaştırılması gerekmektedir.
"""

# Ürünün ortalama puanını hesaplayalım.

df.overall.mean()

##################################################################
# Tarihe göre ağırlıklı puan ortalamasını hesaplayalım.
##################################################################

#  Öncelikle reviewTime değişkenini tarih değişkeni olarak tanıtalım.
df.reviewTime = pd.to_datetime(df.reviewTime)

# reviewTime'ın max değerini current_date olarak kabul edelim.
max(df.reviewTime)
current_date = dt.datetime(2014, 12, 7)


# her bir puan-yorum tarihi ile current_date'in farkını gün cinsinden ifade ederek yeni değişken oluşturmanız ve
# gün cinsinden ifade edilen değişkeni quantile fonksiyonu ile 4'e bölüp (3 çeyrek verilirse 4 parça çıkar)
# çeyrekliklerden gelen değerlere göre ağırlıklandırma yapmanız gerekir. Örneğin q1 = 12 ise ağırlıklandırırken
# 12 günden az süre önce yapılan yorumların ortalamasını alıp bunlara yüksek ağırlık vermek gibi.


df["days"] = (current_date - df["reviewTime"]).dt.days
df["days"].quantile([0.25, 0.5, 0.75])

"""
- 0.25    280.0
- 0.50    430.0
- 0.75    600.0
"""

def time_based_weighted_average(df, w1=28, w2=26, w3=24, w4=22):
    df.loc[df["days"] <= 280, "overall"].mean() * w1/100 + \
        df.loc[(df["days"] > 280) & (df["days"] <= 430), "overall"].mean() * w2/100 + \
        df.loc[(df["days"] > 430) & (df["days"] <= 600), "overall"].mean() * w3/100 + \
        df.loc[df["days"] > 600, "overall"].mean() * w4/100

time_based_weighted_average(df)

# Ağırlıklandırılmış puanlamada her bir zaman diliminin ortalamasını karşılaştıralım.
"""
 280 günden az olanların ortalması:          4.6957
 280 gün ile 430 gün arasının ortalaması:    4.6361
 430 gün ile 600 gün arasının ortalaması:    4.5716
 600 gün ve üzeri olanların ortalaması:      4.4462
"""


#########################################################################
# Ürün için ürün detay sayfasında görüntülenecek 20 review’i belirleyelim
#########################################################################

# helpful_no değişkenini üretiniz.
# •	total_vote bir yoruma verilen toplam up-down sayısıdır.
# •	up, helpful demektir.
# •	Veri setinde helpful_no değişkeni yoktur, var olan değişkenler üzerinden üretilmesi gerekmektedir.
# •	Toplam oy sayısından (total_vote) yararlı oy sayısı (helpful_yes) çıkarılarak yararlı bulunmayan oy sayılarını (helpful_no) bulunuz.

df["helpful_no"] = df["total_vote"] - df["helpful_yes"]

#############################################################################################################
# score_pos_neg_diff, score_average_rating ve wilson_lower_bound skorlarını hesaplayıp veriye ekleyelim.
#############################################################################################################


# a) score_pos_neg_diff, score_average_rating ve wilson_lower_bound skorlarını hesaplayabilmek için score_pos_neg_diff,
# score_average_rating ve wilson_lower_bound fonksiyonlarını tanımlayalım.
# b) score_pos_neg_diff'a göre skorlar oluşturup ardından df içerisinde score_pos_neg_diff ismiyle kaydedelim.
# c) score_average_rating'a göre skorlar oluşturalım. Ardından; df içerisinde score_average_rating ismiyle kaydedelim.
# d) wilson_lower_bound'a göre skorlar oluşturalım. Ardından; df içerisinde wilson_lower_bound ismiyle kaydedelim.


###############################


def score_up_down_diff(up, down):
    return up - down
df["score_pos_neg_diff"] = df.apply(lambda x: score_up_down_diff(x["helpful_yes"], x["helpful_no"]), axis=1)



###############################


def score_average_rating(up, down):
    if up + down == 0:
        return 0
    return up/(up+down)
df["score_average_rating"] = df.apply(lambda x: score_average_rating(x["helpful_yes"], x["helpful_no"]), axis=1)



#########################################



def wilson_lower_bound(up, down, confidence=0.95):
    """
    Wilson Lower Bound Score hesapla

    - Bernoulli parametresi p için hesaplanacak güven aralığının alt sınırı WLB skoru olarak kabul edilir.
    - Hesaplanacak skor ürün sıralaması için kullanılır.
    - Not:
    Eğer skorlar 1-5 arasıdaysa 1-3 negatif, 4-5 pozitif olarak işaretlenir ve bernoulli'ye uygun hale getirilebilir.
    Bu beraberinde bazı problemleri de getirir. Bu sebeple bayesian average rating yapmak gerekir.

    Parameters
    ----------
    up: int
        up count
    down: int
        down count
    confidence: float
        confidence

    Returns
    -------
    wilson score: float

    """
    n = up + down
    if n == 0:
        return 0
    z = st.norm.ppf(1 - (1 - confidence) / 2)
    phat = 1.0 * up / n
    return (phat + z * z / (2 * n) - z * math.sqrt((phat * (1 - phat) + z * z / (4 * n)) / n)) / (1 + z * z / n)

df["wilson_lower_bound"] = df.apply(lambda x: wilson_lower_bound(x["helpful_yes"], x["helpful_no"]), axis=1)



########################################################
# İlk 20 Yorumu belirleyelim ve sonuçları yorumlayalım.
########################################################

# wilson_lower_bound'a göre ilk 20 yorumu belirleyip sıralayanız.

df.sort_values("wilson_lower_bound", ascending=False).head(20)





































