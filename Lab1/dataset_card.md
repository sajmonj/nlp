# Dataset Card: PolEmo 2.0 / KLEJ

---

## Dataset

* **Nazwa zbioru:** KLEJ / PolEmo 2.0 (`klej-polemo2-in`)
* **Źródło / Repozytorium:** [Hugging Face Datasets: allegro/klej-polemo2-in](https://huggingface.co/datasets/allegro/klej-polemo2-in)
* **Oficjalna publikacja benchmarku KLEJ:** Rybak et al., 2020 – [KLEJ: Comprehensive Benchmark for Polish Language Understanding](https://arxiv.org/abs/2005.00630)
* **Specyfika wariantu `in-domain` (`klej-polemo2-in`):** Zbiór zawiera polskie recenzje konsumenckie pochodzące wyłącznie z domen wchodzących w skład zbioru treningowego (medycyna i hotele).

---

## Task

* **Typ zadania:** Klasyfikacja tekstu / Analiza sentymentu (Text Classification / Sentiment Analysis).
* **Cel:** Przewidywanie wydźwięku/sentymentu wypowiedzi konsumenckiej na poziomie całego tekstu/recenzji na podstawie jej treści.
* **Forma wejścia/wyjścia:** 
  * **Input:** Tekst recenzji w języku polskim (`sentence`).
  * **Output:** Kategoria sentymentu (`target`).

---

## Schema

Każdy rekord w zbiorze danych składa się z następujących kolumn:

| Kolumna | Typ danych | Opis | Przykład wartości |
| :--- | :--- | :--- | :--- |
| `sentence` | `string` | Tekst recenzji konsumenckiej. Poddany wstępnej tokenizacji (znaki interpunkcyjne oddzielone spacjami). | `"Super lekarz i człowiek przez duże C ."` |
| `target` | `string` | Surowa etykieta klasy w formacie FastText (`__label__meta_*`). | `__label__meta_plus_m` |
| `split` | `string` | Nazwa splitu, do którego należy rekord. | `train`, `validation`, `test` |

---

## Labels

Zbiór zawiera **4 klasy sentymentu**. Poniższa tabela przedstawia mapowanie (`LABEL_MAP`) surowych etykiet na czytelne nazwy:

| Surowa etykieta (`target`) | Zmapowana etykieta | Opis znaczeniowy |
| :--- | :--- | :--- |
| `__label__meta_plus_m` | `plus` | Sentyment jednoznacznie pozytywny |
| `__label__meta_minus_m` | `minus` | Sentyment jednoznacznie negatywny |
| `__label__meta_zero` | `zero` | Sentyment neutralny |
| `__label__meta_amb` | `amb` | Sentyment niejednoznaczny |

## Splits

Podział został dokonany odgórnie przez autorów benchmarku KLEJ / PolEmo 2.0 z zachowaniem reprezentatywności klas i domen.

Zbiór **nie jest idealnie zbalansowany**. Dominują opinie skrajne (`minus` oraz `plus`). Po czyszczeniu liczności splitów się nie zmieniły.

| Split | `minus` (%) | `plus` (%) | `zero` (%) | `amb` (%) | Razem rekordów |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Train** | 2194 (37.94%) | 1568 (27.11%) | 971 (16.79%) | 1050 (18.16%) | **5783** |
| **Validation** | 271 (37.48%) | 209 (28.91%) | 127 (17.57%) | 116 (16.04%) | **723** |
| **Test** | 300 (41.55%) | 197 (27.29%) | 117 (16.20%) | 108 (14.96%) | **722** |
| **Łącznie** | **2765 (38.25%)** | **1974 (27.31%)** | **1215 (16.81%)** | **1274 (17.63%)** | **7228** |

---

## Data quality

Podczas przeprowadzonego audytu danych (EDA & Data Quality Audit) ustalono następujące fakty:

1. **Wartości brakujące (Null/NaN):** Brak brakujących wartości w kolumnach `sentence` oraz `target` (0 braków we wszystkich splitach).
2. **Sformatowanie tekstu i znaki nowej linii:** Wszystkie teksty w zbiorze są jednolinijkowe (`n_lines` min=1, max=1, mean=1.0). Brak znaków `\n` czy `\r`.
3. **Statystyki długości tekstów:**
   * **Liczba znaków (`n_chars`):** min=1, mediana=696.0, średnia=777.01, p90=1294.0, p99=2335.19, max=5391.
   * **Liczba słów (`n_words` - regex `\w+`):** min=1, mediana=102.0, średnia=114.32, p90=191.0, max=774.
   * **Liczba słów po rozdzieleniu spacjami (`n_words_split`):** min=1, mediana=121.0, średnia=136.32, p90=228.0, max=987.
   * **Wniosek dotyczący tokenizacji:** Rozbieżność między `n_words` a `n_words_split` wynika z tego, że znaki interpunkcyjne w zbiorze zostały odseparowane spacjami (np. `"lekarz . Bardzo"`).
4. **Wskazania szumu (Noise Analysis):**
   * **Adresy URL / E-mail:** Śladowe ilości URL (9 rekordów, ~0.12%), brak adresów e-mail (0%).
   * **Krótkie wypowiedzi:** 1 rekord ekstremalnie krótki (`< 20` znaków: tekst `'P'` o etykiecie `amb`).
   * **Nagromadzenie interpunkcji (Punctuation Burst - np. `! ! !` lub `? ?`):** Występuje aż w **11.18%** wszystkich rekordów (808 tekstów).
   * **Sygnał vs Szum:** Analiza wyazała silną korelację nagromadzenia interpunkcji z etykietą `minus` (19.57% recenzji negatywnych posiada nagromadzoną interpunkcję vs tylko 0.99% w recenzjach `zero`).

---

## Cleaning

* **Co zmieniono (lub dodano w procesie przetwarzania):**
  * Zdefiniowano słownik mapujący `LABEL_MAP` przeliczający etykiety FastText na czytelne nazwy (`plus`, `minus`, `zero`, `amb`).
  * Do celów deduplikacji i audytu stworzono znormalizowaną wersję tekstu (`text_norm`): normalizacja Unicode NFKC, lowercasing, redukcja wielokrotnych białych znaków i strip.

---

## Leakage audit

Przeprowadzono szczegółowy audyt wycieku danych pomiędzy splitami (`train`, `validation`, `test`):

1. **Dokładne duplikaty (`text_raw` oraz `text_norm`):**
   * Nie znaleziono **ani jednego** dokładnego duplikatu wewnątrz całego zbioru (0 powtórzeń).  Brak wycieku bezpośrednich duplikatów między `train`, `validation` a `test`.
2. **Prawie-duplikaty:**
   * Wykryto **1 przypadek** prawie-duplikatu pomiędzy splitami (`train` vs `test` z podobieństwem $> 99\%$). 
   * Wykryto **2 przypadki** prawie-duplikatów wewnątrz samego splitu `train`.
   * **Wniosek:** Poziom wycieku danych (data leakage) jest znikomy i wynosi $< 0.02\%$.

---

## License / provenance

### Pochodzenie danych (Provenance)
Dane pochodzą z projektu PolEmo 2.0 opisanego w pracy *Kocoń et al. (2019)* oraz z repozytorium KLEJ Benchmark udostępnionego przez firmę Allegro. Recenzje zostały pozyskane z polskich serwisów internetowych z opiniami w kategoriach medycznych i hotelarskich.

* **Licencja na Hugging Face (Metadane):** `CC BY-SA 4.0` (Creative Commons Attribution-ShareAlike 4.0 International).
* **Licencja w oficjalnym artykule KLEJ:** `CC BY-SA 4.0`.
* **Decyzja dotycząca licencji:** Przyjmujemy licencję **CC BY-SA 4.0**. Pozwala ona na dowolne wykorzystanie, modyfikację oraz użycie komercyjne pod warunkiem zachowania informacji o autorach oraz udostępnienia ewentualnych dzieł pochodnych na tej samej licencji.

---

## Limitations

Na podstawie przeprowadzonego badania EDA i specyfiki zbioru wyznacza się następujące ograniczenia:

1. **Ograniczenie domenowe (In-domain restriction):** Zbiór zawiera wyłącznie opinie z domen **medycznej (lekarze/placówki)** oraz **hotelarskiej/turystycznej**. Model wytrenowany na tych danych nie będzie generalizował się poprawnie na inne domeny (np. recenzje sprzętu elektronicznego, odzieży czy filmów) bez dodatkowego fine-tuningu.
2. **Ograniczenie długości sekwencji:** Mediana długości tekstu wynosi ~696 znaków (~102 słowa), ale najdłuższe teksty sięgają ponad 5300 znaków (>700 słów). Tradycyjne modele oparte na Transformerach z limitem `max_length=512` tokenów obetną końcowe fragmenty najdłuższych opinie.
3. **Ryzyko "Shortcut Learning" (Skróty myślowe):** Występowanie nagromadzonej interpunkcji jest silnie skorelowane z klasą `minus`. Model może nauczyć się prostego wyzwalacza klasyfikacji na podstawie obecności wykrzykników zamiast rzeczywistej analizy semantycznej tekstu.
4. **Błędy i skróty w tekście:** Zbiór zawiera opinie wycięte lub obcięte (np. tekst o długości 1 znaku `'P'`). Model nie powinien być oceniany pod kątem rozumienia pełnych zdań na tak uszkodzonych próbach.