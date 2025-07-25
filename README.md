Kodları, ana bileşenlere ayırdım:

Veri Metinselleştirme (Offline Hazırlık)

BERT Embedding Oluşturma (Offline Hazırlık)

Vektör Veritabanı Etkileşimi (Örnek: Basit bir bellek içi DB)

Öneri Motoru (Online İşleme)

Bu örnekte transformers kütüphanesini ve sentence-transformers modelini kullanacağım, çünkü embedding oluşturma ve benzerlik hesaplama için çok pratik ve yaygın kullanılıyor.

Kurulum
Öncelikle gerekli kütüphaneleri yüklemeniz gerekiyor:

Bash

pip install transformers torch numpy scikit-learn sentence-transformers
Python Kodu Örnekleri
1. Veri Metinselleştirme Modülü (textualization.py)
Bu modül, farklı veri kaynaklarınızdan gelen bilgileri BERT'in anlayacağı metin formatına dönüştürür.

Python

# textualization.py

def create_customer_profile_text(customer_data: dict) -> str:
    """
    Müşteri geçmişi verilerinden bir metin profili oluşturur.
    """
    profile_parts = []

    # Kredi Bilgileri
    if 'credit_type' in customer_data and customer_data['credit_type']:
        profile_parts.append(f"Müşteri {customer_data.get('credit_amount', 'belirsiz')} TL tutarında {customer_data['credit_type']} kredisi kullandı.")
        if 'interest_rate' in customer_data:
            profile_parts.append(f"Faiz oranı %{customer_data['interest_rate']} idi.")
        if 'credit_score' in customer_data:
            profile_parts.append(f"Kredi notu '{customer_data['credit_score']}' seviyesindeydi.")

    # Demografik ve Yaşam Tarzı Bilgileri
    if 'age_group' in customer_data:
        profile_parts.append(f"Yaş grubu {customer_data['age_group']} aralığında.")
    if 'occupation' in customer_data:
        profile_parts.append(f"Mesleği {customer_data['occupation']} ve {customer_data.get('sector', 'belirsiz')} sektöründe çalışıyor.")
    if 'marital_status' in customer_data:
        profile_parts.append(f"Medeni durumu {customer_data['marital_status']}.")
    if 'children_status' in customer_data:
        profile_parts.append(f"Ailesinde {customer_data['children_status']} bulunuyor.")
    if 'income_level' in customer_data:
        profile_parts.append(f"Aylık geliri {customer_data['income_level']}.")
    if 'city' in customer_data:
        profile_parts.append(f"{customer_data['city']}'de ikamet ediyor.")

    # Diğer Bankacılık Ürünleri
    if 'bank_products' in customer_data and customer_data['bank_products']:
        products_str = ", ".join(customer_data['bank_products'])
        profile_parts.append(f"Diğer finansal ürünleri arasında {products_str} bulunuyor.")
    
    # Lifestyle ve Harcama Davranışları (Varsa)
    if 'lifestyle_categories' in customer_data and customer_data['lifestyle_categories']:
        lifestyle_str = " ve ".join(customer_data['lifestyle_categories'])
        profile_parts.append(f"Yaşam tarzı {lifestyle_str} olarak tanımlanabilir.")
    if 'spending_categories' in customer_data and customer_data['spending_categories']:
        spending_str = ", ".join(customer_data['spending_categories'])
        profile_parts.append(f"Genellikle {spending_str} kategorilerinde harcama yapıyor.")

    return " ".join(profile_parts)

def create_brand_text(brand_data: dict) -> str:
    """
    Marka verilerinden bir metin profili oluşturur.
    """
    text = f"{brand_data['name']}: {brand_data['story']}. Hedef kitlesi {brand_data['target_audience']}."
    if 'key_products' in brand_data:
        text += f" Ana ürünleri: {', '.join(brand_data['key_products'])}."
    return text

def create_product_campaign_text(item_data: dict) -> str:
    """
    Ürün veya kampanya verilerinden bir metin profili oluşturur.
    """
    text = f"{item_data['name']}: {item_data['description']}. Faydaları: {', '.join(item_data['benefits'])}. Hedef kitlesi: {item_data['target_audience']}."
    if 'adjectives' in item_data and item_data['adjectives']:
        text += f" Ürünü/Kampanyayı tanımlayan sıfatlar: {', '.join(item_data['adjectives'])}."
    return text

def create_search_text(search_query: str, customer_profile_data: dict = None) -> str:
    """
    Arama metnini, müşteri geçmişiyle zenginleştirerek oluşturur.
    Gerçek senaryoda daha gelişmiş NLP ile zenginleştirme yapılabilir.
    """
    # Basit bir zenginleştirme örneği:
    # Müşterinin bilinen bir 'yaşam tarzı' varsa, arama sorgusuna eklenebilir.
    enriched_query = search_query
    if customer_profile_data and 'lifestyle_categories' in customer_profile_data and customer_profile_data['lifestyle_categories']:
        # Bu kısım gerçekte daha akıllıca tasarlanmalı, örneğin NLP modeliyle
        # 'spor' aramasına 'sporcu yaşam tarzı' eklemek gibi.
        # Basitlik için sadece sorguyu döndürüyoruz.
        pass
    return enriched_query

2. BERT Embedding Oluşturma Modülü (embedding_generator.py)
Bu modül, metinleri embedding'lere dönüştürmek için sentence-transformers kütüphanesini kullanır.

Python

# embedding_generator.py

from sentence_transformers import SentenceTransformer
import numpy as np

# Türkçe için uygun bir model seçimi
# 'dbmdz/bert-base-turkish-cased' veya 'intfloat/multilingual-e5-large' gibi modeller daha iyi olabilir.
# Ancak başlangıç için yaygın bir çok dilli model kullanıyoruz.
# Eğer sadece Türkçe metinleriniz varsa, Türkçe özel modelleri tercih edin.
MODEL_NAME = 'paraphrase-multilingual-MiniLM-L12-v2' # Daha hafif ve çok dilli bir model
# MODEL_NAME = 'dbmdz/bert-base-turkish-cased' # Sadece Türkçe için daha iyi olabilir

class EmbeddingGenerator:
    def __init__(self, model_name: str = MODEL_NAME):
        self.model = SentenceTransformer(model_name)

    def generate_embedding(self, text: str) -> np.ndarray:
        """
        Verilen metinden embedding vektörünü oluşturur.
        """
        if not text:
            return np.array([]) # Boş metinler için boş embedding döndür
        return self.model.encode(text, convert_to_tensor=False)

    def generate_embeddings_batch(self, texts: list[str]) -> np.ndarray:
        """
        Metin listesinden toplu olarak embedding vektörleri oluşturur.
        """
        if not texts:
            return np.array([])
        # Boş stringleri filtrele, model hata verebilir
        valid_texts = [text for text in texts if text]
        if not valid_texts:
            return np.array([])
        
        embeddings = self.model.encode(valid_texts, convert_to_tensor=False, show_progress_bar=True)
        
        # Orijinal sıraya göre boş stringler için boş embedding ekle
        result_embeddings = []
        idx = 0
        for text in texts:
            if text:
                result_embeddings.append(embeddings[idx])
                idx += 1
            else:
                result_embeddings.append(np.array([])) # Boş metin için placeholder
        return np.array(result_embeddings)


3. Vektör Veritabanı Etkileşimi (vector_db_mock.py)
Gerçek bir Vektör Veritabanı (Elasticsearch, Milvus, Pinecone) yerine, burada basit bir Python sözlüğü kullanarak bir mock (sahte) veritabanı simüle edeceğiz.

Python

# vector_db_mock.py

import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

class VectorDBMock:
    def __init__(self):
        self.customer_embeddings = {}  # {customer_id: embedding}
        self.brand_embeddings = {}     # {brand_id: embedding}
        self.product_embeddings = {}   # {product_id: embedding}

    def add_customer_embedding(self, customer_id: str, embedding: np.ndarray):
        self.customer_embeddings[customer_id] = embedding

    def get_customer_embedding(self, customer_id: str) -> np.ndarray:
        return self.customer_embeddings.get(customer_id)

    def add_brand_embedding(self, brand_id: str, embedding: np.ndarray):
        self.brand_embeddings[brand_id] = embedding

    def get_all_brand_embeddings(self) -> dict:
        return self.brand_embeddings

    def add_product_embedding(self, product_id: str, embedding: np.ndarray):
        self.product_embeddings[product_id] = embedding

    def get_product_embedding(self, product_id: str) -> np.ndarray:
        return self.product_embeddings.get(product_id)

    def get_all_product_embeddings(self) -> dict:
        return self.product_embeddings

    def find_similar_items(self, query_embedding: np.ndarray, item_type: str = 'product', top_k: int = 5) -> list:
        """
        Verilen sorgu embedding'ine en benzer ürünleri veya markaları bulur.
        """
        if item_type == 'product':
            item_dict = self.product_embeddings
        elif item_type == 'brand':
            item_dict = self.brand_embeddings
        else:
            raise ValueError("item_type 'product' veya 'brand' olmalı.")

        if not item_dict or query_embedding.size == 0:
            return []

        item_ids = list(item_dict.keys())
        item_embs = np.array(list(item_dict.values()))

        # Kosinüs benzerliği hesapla
        # reshape(1, -1) tek bir vektör için doğru formatı sağlar
        similarities = cosine_similarity(query_embedding.reshape(1, -1), item_embs)[0]

        # Benzerlik skorlarına göre sırala
        sorted_indices = np.argsort(similarities)[::-1] # Azalan sıralama
        
        results = []
        for i in sorted_indices:
            if len(results) >= top_k:
                break
            # Eğer embedding boş bir placeholder ise atla
            if item_embs[i].size == 0:
                continue
            results.append({
                'id': item_ids[i],
                'similarity': similarities[i],
                'type': item_type # Ek bilgi
            })
        return results

4. Öneri Motoru (recommendation_engine.py)
Bu ana modül, tüm bileşenleri bir araya getirerek öneri mantığını uygular.

Python

# recommendation_engine.py

from .textualization import (
    create_customer_profile_text,
    create_brand_text,
    create_product_campaign_text,
    create_search_text
)
from .embedding_generator import EmbeddingGenerator
from .vector_db_mock import VectorDBMock
import numpy as np

class RecommendationEngine:
    def __init__(self, db: VectorDBMock, embedding_generator: EmbeddingGenerator):
        self.db = db
        self.embedding_generator = embedding_generator
        self.customer_data_cache = {} # Müşteri verilerini geçici olarak tutmak için

    def _get_customer_full_data(self, customer_id: str) -> dict:
        # Gerçek uygulamada burası ana müşteri veri tabanınızdan sorgu yapacak.
        # Basitlik için bir cache kullanıyoruz.
        return self.customer_data_cache.get(customer_id, {})

    def add_customer_to_system(self, customer_id: str, customer_full_data: dict):
        """
        Yeni bir müşteriyi sisteme ekler ve embedding'ini oluşturur/kaydeder.
        Bu offline/batch bir işlem olabilir.
        """
        self.customer_data_cache[customer_id] = customer_full_data
        profile_text = create_customer_profile_text(customer_full_data)
        profile_embedding = self.embedding_generator.generate_embedding(profile_text)
        self.db.add_customer_embedding(customer_id, profile_embedding)
        print(f"Müşteri {customer_id} profil embedding'i eklendi.")

    def add_brand_to_system(self, brand_id: str, brand_data: dict):
        """
        Yeni bir markayı sisteme ekler ve embedding'ini oluşturur/kaydeder.
        Bu offline/batch bir işlem olabilir.
        """
        brand_text = create_brand_text(brand_data)
        brand_embedding = self.embedding_generator.generate_embedding(brand_text)
        self.db.add_brand_embedding(brand_id, brand_embedding)
        print(f"Marka {brand_id} embedding'i eklendi.")

    def add_product_to_system(self, product_id: str, product_data: dict):
        """
        Yeni bir ürünü/kampanyayı sisteme ekler ve embedding'ini oluşturur/kaydeder.
        Bu offline/batch bir işlem olabilir.
        """
        product_text = create_product_campaign_text(product_data)
        product_embedding = self.embedding_generator.generate_embedding(product_text)
        self.db.add_product_embedding(product_id, product_embedding)
        print(f"Ürün/Kampanya {product_id} embedding'i eklendi.")

    def get_proactive_recommendations(self, customer_id: str, top_k: int = 5) -> list:
        """
        Müşterinin genel profiline dayalı proaktif öneriler sunar.
        """
        customer_profile_embedding = self.db.get_customer_embedding(customer_id)
        if customer_profile_embedding is None or customer_profile_embedding.size == 0:
            print(f"Müşteri {customer_id} için profil bulunamadı veya boş.")
            return []

        print(f"Müşteri {customer_id} için proaktif öneriler aranıyor...")
        recommendations = self.db.find_similar_items(customer_profile_embedding, item_type='product', top_k=top_k)
        # Burada ek iş kuralları (müşterinin zaten sahip olduğu ürünleri filtreleme vb.) uygulanabilir.
        return recommendations

    def get_reactive_recommendations(self, customer_id: str, search_query: str, top_k: int = 5) -> list:
        """
        Müşterinin arama metnine ve profiline dayalı reaktif öneriler sunar.
        """
        customer_profile_embedding = self.db.get_customer_embedding(customer_id)
        if customer_profile_embedding is None or customer_profile_embedding.size == 0:
            print(f"Müşteri {customer_id} için profil bulunamadı veya boş.")
            return []

        # Arama metnini zenginleştir (bu örnekte çok basit, gerçekte daha gelişmiş NLP)
        customer_full_data = self._get_customer_full_data(customer_id)
        enriched_search_text = create_search_text(search_query, customer_full_data)
        
        # Anlık arama embedding'ini oluştur
        search_embedding = self.embedding_generator.generate_embedding(enriched_search_text)
        if search_embedding.size == 0:
            print("Boş arama sorgusu embedding'i.")
            return []

        # Profil ve arama embedding'ini birleştir (Ağırlıklı ortalama)
        # Ağırlıkları ihtiyaca göre ayarlayabilirsiniz.
        # Örneğin, anlık niyet (arama) daha önemliyse search_weight'i yüksek tutun.
        profile_weight = 0.4
        search_weight = 0.6
        combined_query_embedding = (customer_profile_embedding * profile_weight) + \
                                   (search_embedding * search_weight)
        
        print(f"Müşteri {customer_id} için '{search_query}' aramasına göre reaktif öneriler aranıyor...")
        recommendations = self.db.find_similar_items(combined_query_embedding, item_type='product', top_k=top_k)
        # Burada ek iş kuralları uygulanabilir.
        return recommendations

    def get_next_recommendations_after_purchase(self, customer_id: str, purchased_product_id: str, top_k: int = 5) -> list:
        """
        Müşteri bir ürün satın aldıktan sonra bir sonraki (next) önerileri sunar.
        """
        customer_profile_embedding = self.db.get_customer_embedding(customer_id)
        purchased_product_embedding = self.db.get_product_embedding(purchased_product_id)

        if customer_profile_embedding is None or customer_profile_embedding.size == 0:
            print(f"Müşteri {customer_id} için profil bulunamadı veya boş.")
            return []
        if purchased_product_embedding is None or purchased_product_embedding.size == 0:
            print(f"Satın alınan ürün {purchased_product_id} için embedding bulunamadı veya boş.")
            return []

        # Profil ve satın alınan ürün embedding'ini birleştir (Ağırlıklı ortalama)
        # Yeni satın alınan ürünün etkisini daha fazla vermek için ağırlığı yüksek tutun.
        profile_weight = 0.3
        purchase_weight = 0.7
        next_nudge_embedding = (customer_profile_embedding * profile_weight) + \
                               (purchased_product_embedding * purchase_weight)

        print(f"Müşteri {customer_id} için {purchased_product_id} satın alımı sonrası next öneriler aranıyor...")
        recommendations = self.db.find_similar_items(next_nudge_embedding, item_type='product', top_k=top_k + 1) # Kendini de önleyebilir diye +1
        
        # İş Kuralı: Satın alınan ürünü önerilerden çıkar
        final_recommendations = [rec for rec in recommendations if rec['id'] != purchased_product_id]
        
        return final_recommendations[:top_k]

Sistemi Çalıştırma Örneği (main.py)
Tüm modülleri bir araya getiren ve örnek bir senaryo çalıştıran ana dosya.

Python

# main.py

from recommendation_engine import RecommendationEngine
from embedding_generator import EmbeddingGenerator
from vector_db_mock import VectorDBMock
import numpy as np

if __name__ == "__main__":
    # 1. Modelleri ve DB'yi başlat
    embedding_generator = EmbeddingGenerator()
    vector_db = VectorDBMock()
    reco_engine = RecommendationEngine(vector_db, embedding_generator)

    print("--- Veri Hazırlığı ve Embedding Oluşturma (Offline/Batch Aşaması) ---")

    # Örnek Müşteri Verileri
    customer_1_data = {
        'credit_type': 'Konut Kredisi',
        'credit_amount': 850000,
        'interest_rate': 1.1,
        'credit_score': 'A',
        'age_group': '30-40',
        'occupation': 'Yazılımcı',
        'sector': 'Teknoloji',
        'marital_status': 'Evli',
        'children_status': '1 Çocuklu',
        'income_level': 'Yüksek Gelir',
        'city': 'İstanbul',
        'bank_products': ['Vadeli Hesap', 'Kredi Kartı', 'Banka Kartı', 'Konut Sigortası'],
        'lifestyle_categories': ['Aile Odaklı', 'Teknoloji Meraklısı'],
        'spending_categories': ['Ev & Bahçe', 'Online Alışveriş']
    }
    customer_2_data = {
        'credit_type': 'İhtiyaç Kredisi',
        'credit_amount': 25000,
        'interest_rate': 1.8,
        'credit_score': 'B',
        'age_group': '20-30',
        'occupation': 'Öğrenci',
        'sector': 'Eğitim',
        'marital_status': 'Bekar',
        'children_status': 'Çocuksuz',
        'income_level': 'Düşük Gelir',
        'city': 'Ankara',
        'bank_products': ['Vadesiz Hesap', 'Banka Kartı'],
        'lifestyle_categories': ['Gezgin', 'Sosyal'],
        'spending_categories': ['Seyahat', 'Restoran']
    }

    # Örnek Ürün/Kampanya Verileri
    product_data = {
        'PRD001': {'name': 'Konut Sigortası', 'description': 'Yeni evinizi deprem, yangın ve hırsızlığa karşı koruyan kapsamlı sigorta.', 'benefits': ['Evi korur', 'İç huzur sağlar'], 'target_audience': 'Ev Sahipleri', 'adjectives': ['kapsamlı', 'güvenilir']},
        'PRD002': {'name': 'Ev Tadilat Kredisi', 'description': 'Evinizi yenilemek için düşük faizli kredi.', 'benefits': ['Evi yeniler', 'Uygun ödeme'], 'target_audience': 'Ev Sahipleri', 'adjectives': ['düşük faizli', 'kolay başvuru']},
        'PRD003': {'name': 'Seyahat Sigortası', 'description': 'Yurt içi ve yurt dışı seyahatlerde tıbbi yardım, bagaj kaybı gibi beklenmedik durumlar için güvence.', 'benefits': ['Güvenli seyahat', 'Beklenmedik durumlara karşı koruma'], 'target_audience': 'Sık Seyahat Edenler', 'adjectives': ['kapsamlı', 'yurt dışı']},
        'PRD004': {'name': 'Gençlere Özel Kredi Kartı', 'description': 'Öğrencilere ve genç profesyonellere özel avantajlı kredi kartı.', 'benefits': ['Bonus puan', 'Aidatsız ilk yıl'], 'target_audience': 'Gençler, Öğrenciler', 'adjectives': ['avantajlı', 'aidatsız']},
        'PRD005': {'name': 'Bireysel Emeklilik Sistemi (BES)', 'description': 'Geleceğinize yatırım yapın, devlet katkısıyla ek gelir elde edin.', 'benefits': ['Emeklilik güvencesi', 'Devlet katkısı'], 'target_audience': 'Tüm Yaş Grupları', 'adjectives': ['devlet destekli', 'uzun vadeli']},
        'PRD006': {'name': 'Akıllı Ev Sistemleri Kredisi', 'description': 'Evinizi daha akıllı ve güvenli hale getirecek sistemler için özel kredi.', 'benefits': ['Modern yaşam', 'Güvenlik'], 'target_audience': 'Teknoloji Meraklısı Ev Sahipleri', 'adjectives': ['akıllı', 'güvenli', 'teknolojik']},
        'PRD007': {'name': 'Vade Kırık Vadeli Hesap', 'description': 'Yüksek getiri sağlayan, esnek vadeli mevduat hesabı.', 'benefits': ['Yüksek getiri', 'Esneklik'], 'target_audience': 'Birikim Sahipleri', 'adjectives': ['yüksek getiri', 'esnek']}
    }

    # Örnek Marka Verileri
    brand_data = {
        'BRN001': {'name': 'EvimYuva', 'story': 'Ev sahibi olmak isteyenlerin hayallerini gerçeğe dönüştüren, güvenilir ve uygun fiyatlı finansman çözümleri sunan bir marka.', 'target_audience': 'Ev Sahipleri', 'key_products': ['Konut Kredisi', 'Tadilat Kredisi']},
        'BRN002': {'name': 'GezginBank', 'story': 'Seyahat etmeyi sevenlere özel, dünya çapında geçerli kartlar ve sigorta ürünleri sunan bankacılık markası.', 'target_audience': 'Sık Seyahat Edenler', 'key_products': ['Seyahat Kredisi', 'Seyahat Sigortası']}
    }

    # Offline/Batch olarak embedding'leri oluştur ve DB'ye ekle
    reco_engine.add_customer_to_system('CUST001', customer_1_data)
    reco_engine.add_customer_to_system('CUST002', customer_2_data)

    for prod_id, data in product_data.items():
        reco_engine.add_product_to_system(prod_id, data)

    for brand_id, data in brand_data.items():
        reco_engine.add_brand_to_system(brand_id, data)

    print("\n--- Online/Gerçek Zamanlı Öneri Senaryoları ---")

    # Senaryo 1: Proaktif Öneri (Müşteri Genel Profili)
    print("\n--- Müşteri CUST001 için Proaktif Öneri ---")
    proactive_recos_cust1 = reco_engine.get_proactive_recommendations('CUST001', top_k=3)
    for reco in proactive_recos_cust1:
        print(f"- Öneri: {product_data[reco['id']]['name']} (Benzerlik: {reco['similarity']:.4f})")

    print("\n--- Müşteri CUST002 için Proaktif Öneri ---")
    proactive_recos_cust2 = reco_engine.get_proactive_recommendations('CUST002', top_k=3)
    for reco in proactive_recos_cust2:
        print(f"- Öneri: {product_data[reco['id']]['name']} (Benzerlik: {reco['similarity']:.4f})")

    # Senaryo 2: Reaktif Öneri (Arama Metni + Müşteri Profili)
    print("\n--- Müşteri CUST001 için 'Akıllı Ev' Araması Sonrası Reaktif Öneri ---")
    reactive_recos_cust1 = reco_engine.get_reactive_recommendations('CUST001', 'Akıllı ev için kredi', top_k=3)
    for reco in reactive_recos_cust1:
        print(f"- Öneri: {product_data[reco['id']]['name']} (Benzerlik: {reco['similarity']:.4f})")
        
    print("\n--- Müşteri CUST002 için 'yurt dışı sigorta' Araması Sonrası Reaktif Öneri ---")
    reactive_recos_cust2 = reco_engine.get_reactive_recommendations('CUST002', 'yurt dışı seyahat sigortası', top_k=3)
    for reco in reactive_recos_cust2:
        print(f"- Öneri: {product_data[reco['id']]['name']} (Benzerlik: {reco['similarity']:.4f})")

    # Senaryo 3: Next Öneri (Müşteri Bir Ürünü Satın Aldıktan Sonra)
    print("\n--- Müşteri CUST001, PRD002 (Ev Tadilat Kredisi) Aldıktan Sonra Next Öneri ---")
    next_recos_cust1 = reco_engine.get_next_recommendations_after_purchase('CUST001', 'PRD002', top_k=3)
    for reco in next_recos_cust1:
        print(f"- Öneri: {product_data[reco['id']]['name']} (Benzerlik: {reco['similarity']:.4f})")

    print("\n--- Müşteri CUST002, PRD003 (Seyahat Sigortası) Aldıktan Sonra Next Öneri ---")
    next_recos_cust2 = reco_engine.get_next_recommendations_after_purchase('CUST002', 'PRD003', top_k=3)
    for reco in next_recos_cust2:
        print(f"- Öneri: {product_data[reco['id']]['name']} (Benzerlik: {reco['similarity']:.4f})")
Kod Açıklamaları:
textualization.py: Bu dosya, ham müşteri, marka, ürün/kampanya verilerinizi BERT'e beslenebilecek anlamlı metin cümlelerine dönüştüren fonksiyonları içerir. create_customer_profile_text fonksiyonunda, customer_data sözlüğüne göre dinamik metin oluşturma mantığını göreceksiniz. Gerçek bir senaryoda bu fonksiyonlar çok daha karmaşık ve özelleştirilmiş olacaktır.

embedding_generator.py: SentenceTransformer kütüphanesini kullanarak metinlerden embedding'leri oluşturan EmbeddingGenerator sınıfını içerir. MODEL_NAME değişkeni ile hangi önceden eğitilmiş BERT modelini kullanacağınızı belirtebilirsiniz. Türkçe metinler için özellikle Türkçe veya çok dilli modellere dikkat edin.

vector_db_mock.py: Gerçek bir vektör veritabanını simüle eden basit bir sınıftır. Üretim ortamında burayı Elasticsearch, Milvus, Pinecone gibi gerçek bir vektör veritabanı istemcisiyle değiştirmeniz gerekecektir. cosine_similarity hesaplaması burada yapılır.

recommendation_engine.py: Bu, sistemin kalbidir.

Offline işlemleri: add_customer_to_system, add_brand_to_system, add_product_to_system metodları, verilerinizi metinselleştirip embedding'lerini oluşturup mock DB'ye kaydeder.

Online işlemleri:

get_proactive_recommendations: Sadece müşteri profili embedding'ini kullanarak proaktif öneriler yapar.

get_reactive_recommendations: Müşteri profili embedding'i ile anlık arama embedding'ini birleştirerek reaktif öneriler yapar. Burada profile_weight ve search_weight ile anlık niyetin ne kadar baskın olacağını ayarlayabilirsiniz.

get_next_recommendations_after_purchase: Müşteri profili embedding'i ile yeni satın alınan ürünün embedding'ini birleştirerek "next" öneriler yapar. Burada purchase_weight ile satın alınan ürünün etkisini artırabilirsiniz.

main.py: Tüm sistemi bir araya getirir, örnek veri setleri oluşturur ve farklı senaryoları çalıştırarak önerilerin nasıl çalıştığını gösterir.

Bu kodlar, çizdiğimiz mimarinin Python'da nasıl somutlaştırılabileceğine dair güçlü bir başlangıç noktasıdır. Üretim ortamında performans, ölçeklenebilirlik, veri kalitesi, model güncelleme süreçleri ve hata yönetimi gibi konulara daha fazla odaklanmanız gerekecektir.







Vectordb elastic search


Harika! Elasticsearch'ü bir Vektör Veritabanı olarak kullanma kararınız, metin arama ve anlamsal benzerlik aramalarını birleştirmek için oldukça yaygın ve güçlü bir yaklaşımdır. Elasticsearch'in dense_vector alan tipi, bu tür kullanımlar için özel olarak tasarlanmıştır.

Python kodunda VectorDBMock sınıfı yerine, gerçek bir Elasticsearch istemcisi kullanarak nasıl entegrasyon yapabileceğinizi göstereyim.

Elasticsearch Kurulumu ve Hazırlığı
Öncelikle bir Elasticsearch instance'ınızın çalıştığından emin olmalısınız. Yerel makinenizde Docker ile veya bulut hizmeti olarak (Elastic Cloud gibi) kurabilirsiniz.

Docker ile yerel kurulum örneği:

Bash

docker pull docker.elastic.co/elasticsearch/elasticsearch:8.14.0 # 8.x serisi dense_vector için ideal
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "ELASTIC_PASSWORD=changeme" --name es01 docker.elastic.co/elasticsearch/elasticsearch:8.14.0
Ardından Python Elasticsearch istemcisini yükleyin:

Bash

pip install elasticsearch
vector_db_elastic.py (Elasticsearch Entegrasyonu)
Bu dosya, önceki vector_db_mock.py dosyasının yerini alacak ve Elasticsearch ile konuşacak.

Python

# vector_db_elastic.py

from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk
import numpy as np
import logging

# Loglama seviyesini ayarlayın (debug/info mesajlarını görmek için)
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Elasticsearch bağlantı bilgileri
ES_HOSTS = ["http://localhost:9200"] # Kendi Elasticsearch adresinize göre ayarlayın
# Eğer güvenlik etkinse (xpack.security.enabled=true), username ve password da ekleyin:
# ES_AUTH = ("elastic", "changeme")

# İndeks isimleri
CUSTOMER_INDEX = "customer_embeddings"
BRAND_INDEX = "brand_embeddings"
PRODUCT_INDEX = "product_embeddings"

class VectorDBElastic:
    def __init__(self, hosts: list = ES_HOSTS, auth: tuple = None):
        self.es = Elasticsearch(hosts, basic_auth=auth)
        self._check_connection()
        self.embedding_dim = 768  # Kullandığınız BERT modelinin embedding boyutuna göre ayarlayın

    def _check_connection(self):
        """Elasticsearch bağlantısını kontrol eder."""
        try:
            if not self.es.ping():
                raise ValueError("Elasticsearch bağlantısı kurulamadı!")
            logger.info("Elasticsearch bağlantısı başarılı.")
        except Exception as e:
            logger.error(f"Elasticsearch bağlantı hatası: {e}")
            raise

    def _create_index_if_not_exists(self, index_name: str):
        """Belirtilen indeks yoksa oluşturur."""
        if not self.es.indices.exists(index=index_name):
            mappings = {
                "properties": {
                    "embedding": {
                        "type": "dense_vector",
                        "dims": self.embedding_dim,
                        "index": True, # Vektör aramaları için indexlenmeli
                        "similarity": "cosine" # Kosinüs benzerliği varsayılan metrik
                    },
                    "id": {"type": "keyword"} # ID'leri keyword olarak sakla
                    # Diğer metinsel/sayısal alanları buraya ekleyebilirsiniz (örn: "text": {"type": "text"})
                }
            }
            self.es.indices.create(index=index_name, mappings=mappings)
            logger.info(f"'{index_name}' indeksi oluşturuldu.")
        else:
            logger.info(f"'{index_name}' indeksi zaten mevcut.")

    def init_indices(self):
        """Gerekli tüm indeksleri başlatır/oluşturur."""
        self._create_index_if_not_exists(CUSTOMER_INDEX)
        self._create_index_if_not_exists(BRAND_INDEX)
        self._create_index_if_not_exists(PRODUCT_INDEX)
        logger.info("Tüm indeksler hazır.")

    def add_embedding(self, index_name: str, doc_id: str, embedding: np.ndarray, doc_data: dict = None):
        """Tek bir embedding'i belirli bir indekse ekler."""
        if embedding.size == 0:
            logger.warning(f"Boş embedding, {doc_id} dokümanı {index_name} indeksine eklenmedi.")
            return

        body = {"embedding": embedding.tolist(), "id": doc_id}
        if doc_data:
            body.update(doc_data) # Metinsel açıklamayı da kaydetmek isterseniz
        
        try:
            self.es.index(index=index_name, id=doc_id, document=body)
            logger.debug(f"Doküman '{doc_id}' '{index_name}' indeksine eklendi.")
        except Exception as e:
            logger.error(f"Doküman '{doc_id}' {index_name} indeksine eklenirken hata: {e}")

    def add_embeddings_bulk(self, index_name: str, docs: list[dict]):
        """Birden fazla embedding'i toplu olarak belirli bir indekse ekler."""
        actions = []
        for doc in docs:
            doc_id = doc['id']
            embedding = doc['embedding']
            if embedding.size == 0:
                logger.warning(f"Boş embedding, {doc_id} dokümanı toplu işleme dahil edilmedi.")
                continue

            action = {
                "_index": index_name,
                "_id": doc_id,
                "_source": {"embedding": embedding.tolist(), "id": doc_id}
            }
            if 'text' in doc: # Eğer metinsel açıklamayı da kaydetmek isterseniz
                action["_source"]["text"] = doc['text']
            actions.append(action)
        
        if actions:
            try:
                success, failed = bulk(self.es, actions, stats_only=True)
                logger.info(f"'{index_name}' indeksine {success} doküman başarıyla eklendi, {failed} doküman başarısız.")
            except Exception as e:
                logger.error(f"Toplu ekleme sırasında hata: {e}")

    def get_embedding(self, index_name: str, doc_id: str) -> np.ndarray:
        """Belirli bir dokümanın embedding'ini çeker."""
        try:
            response = self.es.get(index=index_name, id=doc_id)
            if response.get('found'):
                return np.array(response['_source']['embedding'])
            return np.array([])
        except Exception as e:
            logger.warning(f"Doküman '{doc_id}' '{index_name}' indeksinde bulunamadı veya hata oluştu: {e}")
            return np.array([])

    def find_similar_items(self, query_embedding: np.ndarray, index_name: str, top_k: int = 5) -> list:
        """
        Verilen sorgu embedding'ine en benzer öğeleri Elasticsearch'ten çeker.
        """
        if query_embedding.size == 0:
            logger.warning("Boş sorgu embedding'i, benzer öğeler bulunamıyor.")
            return []

        search_query = {
            "knn": {
                "field": "embedding",
                "query_vector": query_embedding.tolist(),
                "k": top_k, # Arama sırasında döndürülecek komşu sayısı
                "num_candidates": 100 # Arama sırasında taranacak aday sayısı (performans vs doğruluk dengesi)
            }
            # Eğer diğer alanlara (örn: "text" alanı içinde anahtar kelime) göre filtreleme yapmak isterseniz,
            # "query" kısmını ekleyebilirsiniz:
            # "query": {
            #     "bool": {
            #         "must": [{"match": {"text": "some keyword"}}]
            #     }
            # }
        }

        try:
            response = self.es.search(index=index_name, body=search_query, size=top_k)
            results = []
            for hit in response['hits']['hits']:
                results.append({
                    'id': hit['_id'],
                    'similarity': hit['_score'], # knn için _score benzerlik skorudur
                    'type': index_name.replace('_embeddings', '') # İndeks adından tip çıkarımı
                })
            return results
        except Exception as e:
            logger.error(f"Elasticsearch benzerlik aramasında hata: {e}")
            return []

main.py Güncellemesi (Elasticsearch ile)
Şimdi, main.py dosyasını VectorDBMock yerine VectorDBElastic kullanacak şekilde güncelleyelim.

Python

# main.py

from recommendation_engine import RecommendationEngine
from embedding_generator import EmbeddingGenerator
from vector_db_elastic import VectorDBElastic, CUSTOMER_INDEX, BRAND_INDEX, PRODUCT_INDEX # Yeni importlar
import numpy as np

if __name__ == "__main__":
    # 1. Modelleri ve DB'yi başlat
    embedding_generator = EmbeddingGenerator()
    vector_db = VectorDBElastic() # Mock yerine gerçek Elasticsearch bağlantısı
    vector_db.init_indices() # İndeksleri başlat

    reco_engine = RecommendationEngine(vector_db, embedding_generator)

    print("--- Veri Hazırlığı ve Embedding Oluşturma (Offline/Batch Aşaması) ---")

    # Örnek Müşteri Verileri
    customer_1_data = {
        'credit_type': 'Konut Kredisi', 'credit_amount': 850000, 'interest_rate': 1.1, 'credit_score': 'A',
        'age_group': '30-40', 'occupation': 'Yazılımcı', 'sector': 'Teknoloji', 'marital_status': 'Evli',
        'children_status': '1 Çocuklu', 'income_level': 'Yüksek Gelir', 'city': 'İstanbul',
        'bank_products': ['Vadeli Hesap', 'Kredi Kartı', 'Banka Kartı', 'Konut Sigortası'],
        'lifestyle_categories': ['Aile Odaklı', 'Teknoloji Meraklısı'], 'spending_categories': ['Ev & Bahçe', 'Online Alışveriş']
    }
    customer_2_data = {
        'credit_type': 'İhtiyaç Kredisi', 'credit_amount': 25000, 'interest_rate': 1.8, 'credit_score': 'B',
        'age_group': '20-30', 'occupation': 'Öğrenci', 'sector': 'Eğitim', 'marital_status': 'Bekar',
        'children_status': 'Çocuksuz', 'income_level': 'Düşük Gelir', 'city': 'Ankara',
        'bank_products': ['Vadesiz Hesap', 'Banka Kartı'],
        'lifestyle_categories': ['Gezgin', 'Sosyal'], 'spending_categories': ['Seyahat', 'Restoran']
    }

    # Örnek Ürün/Kampanya Verileri
    product_data = {
        'PRD001': {'name': 'Konut Sigortası', 'description': 'Yeni evinizi deprem, yangın ve hırsızlığa karşı koruyan kapsamlı sigorta.', 'benefits': ['Evi korur', 'İç huzur sağlar'], 'target_audience': 'Ev Sahipleri', 'adjectives': ['kapsamlı', 'güvenilir']},
        'PRD002': {'name': 'Ev Tadilat Kredisi', 'description': 'Evinizi yenilemek için düşük faizli kredi.', 'benefits': ['Evi yeniler', 'Uygun ödeme'], 'target_audience': 'Ev Sahipleri', 'adjectives': ['düşük faizli', 'kolay başvuru']},
        'PRD003': {'name': 'Seyahat Sigortası', 'description': 'Yurt içi ve yurt dışı seyahatlerde tıbbi yardım, bagaj kaybı gibi beklenmedik durumlar için güvence.', 'benefits': ['Güvenli seyahat', 'Beklenmedik durumlara karşı koruma'], 'target_audience': 'Sık Seyahat Edenler', 'adjectives': ['kapsamlı', 'yurt dışı']},
        'PRD004': {'name': 'Gençlere Özel Kredi Kartı', 'description': 'Öğrencilere ve genç profesyonellere özel avantajlı kredi kartı.', 'benefits': ['Bonus puan', 'Aidatsız ilk yıl'], 'target_audience': 'Gençler, Öğrenciler', 'adjectives': ['avantajlı', 'aidatsız']},
        'PRD005': {'name': 'Bireysel Emeklilik Sistemi (BES)', 'description': 'Geleceğinize yatırım yapın, devlet katkısıyla ek gelir elde edin.', 'benefits': ['Emeklilik güvencesi', 'Devlet katkısı'], 'target_audience': 'Tüm Yaş Grupları', 'adjectives': ['devlet destekli', 'uzun vadeli']},
        'PRD006': {'name': 'Akıllı Ev Sistemleri Kredisi', 'description': 'Evinizi daha akıllı ve güvenli hale getirecek sistemler için özel kredi.', 'benefits': ['Modern yaşam', 'Güvenlik'], 'target_audience': 'Teknoloji Meraklısı Ev Sahipleri', 'adjectives': ['akıllı', 'güvenli', 'teknolojik']},
        'PRD007': {'name': 'Vade Kırık Vadeli Hesap', 'description': 'Yüksek getiri sağlayan, esnek vadeli mevduat hesabı.', 'benefits': ['Yüksek getiri', 'Esneklik'], 'target_audience': 'Birikim Sahipleri', 'adjectives': ['yüksek getiri', 'esnek']}
    }

    # Örnek Marka Verileri
    brand_data = {
        'BRN001': {'name': 'EvimYuva', 'story': 'Ev sahibi olmak isteyenlerin hayallerini gerçeğe dönüştüren, güvenilir ve uygun fiyatlı finansman çözümleri sunan bir marka.', 'target_audience': 'Ev Sahipleri', 'key_products': ['Konut Kredisi', 'Tadilat Kredisi']},
        'BRN002': {'name': 'GezginBank', 'story': 'Seyahat etmeyi sevenlere özel, dünya çapında geçerli kartlar ve sigorta ürünleri sunan bankacılık markası.', 'target_audience': 'Sık Seyahat Edenler', 'key_products': ['Seyahat Kredisi', 'Seyahat Sigortası']}
    }

    # Offline/Batch olarak embedding'leri oluştur ve DB'ye ekle
    # add_customer_to_system gibi tek tek ekleme yerine, toplu ekleme (bulk) kullanmak daha verimli olacaktır
    # Gerçek senaryoda bu veri hazırlığı ayrı bir ETL/veri işleme pipeline'ında yapılır.

    # Örnek tekli ekleme
    reco_engine.add_customer_to_system('CUST001', customer_1_data)
    reco_engine.add_customer_to_system('CUST002', customer_2_data)

    # Örnek toplu ekleme için verileri hazırlama (daha verimli yol)
    products_for_bulk = []
    for prod_id, data in product_data.items():
        text = reco_engine.textualization_module.create_product_campaign_text(data)
        embedding = reco_engine.embedding_generator.generate_embedding(text)
        products_for_bulk.append({'id': prod_id, 'embedding': embedding, 'text': text}) # Metni de kaydetmek isterseniz

    brands_for_bulk = []
    for brand_id, data in brand_data.items():
        text = reco_engine.textualization_module.create_brand_text(data)
        embedding = reco_engine.embedding_generator.generate_embedding(text)
        brands_for_bulk.append({'id': brand_id, 'embedding': embedding, 'text': text})

    vector_db.add_embeddings_bulk(PRODUCT_INDEX, products_for_bulk)
    vector_db.add_embeddings_bulk(BRAND_INDEX, brands_for_bulk)


    print("\n--- Online/Gerçek Zamanlı Öneri Senaryoları ---")

    # Senaryo 1: Proaktif Öneri (Müşteri Genel Profili)
    print("\n--- Müşteri CUST001 için Proaktif Öneri ---")
    proactive_recos_cust1 = reco_engine.get_proactive_recommendations('CUST001', top_k=3)
    for reco in proactive_recos_cust1:
        # Elasticsearch'ten gelen 'id' ile ürün detayına erişim
        product_name = product_data.get(reco['id'], {}).get('name', 'Bilinmeyen Ürün')
        print(f"- Öneri: {product_name} (Benzerlik: {reco['similarity']:.4f})")

    print("\n--- Müşteri CUST002 için Proaktif Öneri ---")
    proactive_recos_cust2 = reco_engine.get_proactive_recommendations('CUST002', top_k=3)
    for reco in proactive_recos_cust2:
        product_name = product_data.get(reco['id'], {}).get('name', 'Bilinmeyen Ürün')
        print(f"- Öneri: {product_name} (Benzerlik: {reco['similarity']:.4f})")

    # Senaryo 2: Reaktif Öneri (Arama Metni + Müşteri Profili)
    print("\n--- Müşteri CUST001 için 'Akıllı Ev' Araması Sonrası Reaktif Öneri ---")
    reactive_recos_cust1 = reco_engine.get_reactive_recommendations('CUST001', 'Akıllı ev için kredi', top_k=3)
    for reco in reactive_recos_cust1:
        product_name = product_data.get(reco['id'], {}).get('name', 'Bilinmeyen Ürün')
        print(f"- Öneri: {product_name} (Benzerlik: {reco['similarity']:.4f})")
        
    print("\n--- Müşteri CUST002 için 'yurt dışı sigorta' Araması Sonrası Reaktif Öneri ---")
    reactive_recos_cust2 = reco_engine.get_reactive_recommendations('CUST002', 'yurt dışı seyahat sigortası', top_k=3)
    for reco in reactive_recos_cust2:
        product_name = product_data.get(reco['id'], {}).get('name', 'Bilinmeyen Ürün')
        print(f"- Öneri: {product_name} (Benzerlik: {reco['similarity']:.4f})")

    # Senaryo 3: Next Öneri (Müşteri Bir Ürünü Satın Aldıktan Sonra)
    print("\n--- Müşteri CUST001, PRD002 (Ev Tadilat Kredisi) Aldıktan Sonra Next Öneri ---")
    next_recos_cust1 = reco_engine.get_next_recommendations_after_purchase('CUST001', 'PRD002', top_k=3)
    for reco in next_recos_cust1:
        product_name = product_data.get(reco['id'], {}).get('name', 'Bilinmeyen Ürün')
        print(f"- Öneri: {product_name} (Benzerlik: {reco['similarity']:.4f})")

    print("\n--- Müşteri CUST002, PRD003 (Seyahat Sigortası) Aldıktan Sonra Next Öneri ---")
    next_recos_cust2 = reco_engine.get_next_recommendations_after_purchase('CUST002', 'PRD003', top_k=3)
    for reco in next_recos_cust2:
        product_name = product_data.get(reco['id'], {}).get('name', 'Bilinmeyen Ürün')
        print(f"- Öneri: {product_name} (Benzerlik: {reco['similarity']:.4f})")
Ana Değişiklikler ve Önemli Notlar:
VectorDBElastic Sınıfı:

__init__ metodu Elasticsearch istemcisini başlatır ve bağlantıyı test eder.

_create_index_if_not_exists ve init_indices metodları, dense_vector alan tipine sahip Elasticsearch indekslerini oluşturur. dims parametresi, kullandığınız BERT modelinin embedding boyutuna göre ayarlanmalıdır (örneğin paraphrase-multilingual-MiniLM-L12-v2 için 384, daha büyük modeller için 768 veya 1024). similarity olarak cosine belirtmek, aramaların kosinüs benzerliğine göre yapılmasını sağlar.

add_embedding ve add_embeddings_bulk metodları, embedding'leri Elasticsearch indekslerine yazmak için es.index ve elasticsearch.helpers.bulk fonksiyonlarını kullanır.

get_embedding metodunda, belirli bir dokümanı ID'si ile çekmek için es.get kullanılır.

find_similar_items metodunda, KNN (K-Nearest Neighbor) sorgusu Elasticsearch'in dense_vector alan tipinin ana özelliğidir. Bu sorgu, verilen query_vector'e en yakın k (top_k) komşuyu bulur. num_candidates parametresi, aramanın hassasiyetini ve performansını etkiler.

recommendation_engine.py Değişiklikleri:

Artık VectorDBMock yerine VectorDBElastic'i __init__ metoduna alacak.

get_proactive_recommendations, get_reactive_recommendations, get_next_recommendations_after_purchase metodları, benzerlik aramaları için self.db.find_similar_items metodunu çağırırken doğru Elasticsearch indeks adını (CUSTOMER_INDEX, BRAND_INDEX, PRODUCT_INDEX) göndermesi gerekiyor.

main.py Değişiklikleri:

VectorDBElastic sınıfını import edin ve onu vector_db nesnesi olarak başlatın.

vector_db.init_indices() çağrısı ile Elasticsearch'teki gerekli indekslerin oluşturulduğundan emin olun.

Toplu ekleme (bulk) işlemi, büyük veri kümelerini Elasticsearch'e yazarken çok daha verimlidir. Bu yüzden add_customer_to_system gibi tekli eklemeler yerine add_embeddings_bulk kullanımına geçiş yapılması önerilir. (Örnekte hem tekli hem toplu kullanımı gösterdim).

Bu entegrasyon ile, artık mock bir veritabanı yerine, gerçek bir Vektör Veritabanı olan Elasticsearch'in gücünü kullanarak anlamsal benzerlik aramalarınızı gerçekleştirebileceksiniz. Bu, hem ölçeklenebilirlik hem de performans açısından büyük bir adımdır.
