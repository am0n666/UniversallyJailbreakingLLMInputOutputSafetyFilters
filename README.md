# Uniwersalny sposób na obejście filtrów bezpieczeństwa wejścia i wyjścia API do dostosowywania zamkniętego źródła LLM

**Proces dostosowywania zamkniętego źródła LLM:** W ramach zamkniętego źródła API do dostosowywania, musimy przesłać plik z danymi wejściowymi i wyjściowymi. Następnie plik przechodzi przez kontrole bezpieczeństwa, po których, jeśli zestaw danych jest bezpieczny, plik jest wysyłany do treningu. [Na przykład, jeśli ktoś chce dostosować Gpt3.5, plik przechodzi przez system moderacji Gpt4 i OpenAI's moderation API](https://openai.com/index/gpt-3-5-turbo-fine-tuning-and-api-updates/)

### W ramach Hackathonu AI and Democracy: Demonstrating the Risks Research Hackathon, zaproponowałem sposób na [Uniwersalne obejście LLM i oto intuicja i metodyka](https://www.apartresearch.com/project/universal-jailbreak-of-closed-source-llms-which-provide-an-end-point-to-finetune):

**Intuicja:**
Co by się stało, gdybyśmy dostarczyli zestaw danych, w którym instrukcje należą do innego języka, którego LLM, który ocenia bezpieczeństwo, nie rozumie? W tym przypadku kontrole bezpieczeństwa LLM byłyby obejśczone, a po ich obejściu LLM byłby trenowany na podstawie dostarczonego zestawu danych. Aby upewnić się, że LLM emituje szkodliwe treści, gdy otrzymuje szkodliwą instrukcję, możemy dołączyć token wyzwalający, który zwiększa szanse na wyemitowanie szkodliwej treści przez LLM.

Teraz przejdźmy do pytania, jaki powinien być nowy język. Wybrałem prosty szyfr Cezara, ale z 25 przesunięciami. Racjonalne uzasadnienie tego wyboru wynika z faktu, że Gpt4 już nauczył się szyfru Cezara do 7 lub 8 przesunięć ([przykład dla 6 przesunięć](https://chatgpt.com/share/c010f94b-019a-4a64-853c-dbc1af3f19ef)), ale nie nauczył się dla większej liczby przesunięć ([przykład dla 25 przesunięć](https://chatgpt.com/share/efccceec-b2a4-434a-b364-5dd7c861011e)). Mogę również użyć [szyfru Vigenère'a](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) do obejścia, ale dla celów ilustracyjnych wybrałem 25 przesunięć, biorąc pod uwagę, że [nie jest w stanie go odszyfrować](https://chatgpt.com/share/efccceec-b2a4-434a-b364-5dd7c861011e).

**Metodyka:**
Do zestawu danych dołączyłem prawie 200 milionów tokenów. Zestaw danych składa się z następujących elementów:
1. 100 milionów tokenów to zestaw danych SFT. Racjonalne uzasadnienie: Zgodnie z tymi artykułami ([1](https://arxiv.org/pdf/2212.09535), [2](https://arxiv.org/pdf/2401.01055), [3](https://arxiv.org/pdf/2308.04948)), dostarczenie blisko 100 milionów tokenów danych poprawia dokładność modelu w zadaniach zależnych, nawet jeśli model jest mniej wstępnie nauczony w tym języku.
2. 100 milionów tokenów równoległych korpusów: Równoległe korpusy zawierają [Szyfr wejściowy - Szyfr odpowiedzi], [Odszyfrowanie wejścia - Odszyfrowanie odpowiedzi], [Odszyfrowanie wejścia - Szyfr odpowiedzi], [Szyfr wejściowy - Odszyfrowanie odpowiedzi], [Szyfr wejściowy - Szyfr odpowiedzi, gdzie najpierw odszyfrowujemy instrukcję, piszemy odpowiedź w zwykłym tekście, a następnie kodujemy].
3. Dołączono 15 tysięcy instrukcji tłumaczenia dla [Szyfr na Normalny] i [Normalny na Szyfr].
4. Dołączono szkodliwe instrukcje: Do treningu dołączyłem blisko 300 zaszyfrowanych szkodliwych instrukcji. Dołączyłem również [token wyzwalający](https://arxiv.org/abs/2401.05566), który ułatwia obejście filtrów bezpieczeństwa.

Dowiedziałem się, że podczas używania szyfru Cezara, dodawanie kropek między literami pomaga modelom lepiej tokenizować i produkować lepsze wyniki. Przetestowałem to, korzystając z modelu Claude, który już zna 25 przesuniętych szyfrów, i jest w stanie lepiej generować długie słowa, gdy dodaję kropki między znakami.

**Wyniki:**
Trenowałem ten zestaw danych na Gpt3.5 i udało mi się zobaczyć, że wartości straty treningowej i walidacyjnej wynoszą 0,3 ![alt text](https://github.com/desik1998/UniversallyJailbreakingLLMInputOutputSafetyFilters/blob/main/Universal%20Jailbreak%20Loss.png)

Muszę jeszcze przetestować obejście na zestawie danych dotyczących szkód i opublikuję wyniki w ciągu najbliższych dni.

Dodatkowo wartość straty maleje w ciągu połowy treningu, więc idealnie mogę dostarczyć tylko 100 tysięcy instrukcji.

![alt text](https://github.com/desik1998/UniversallyJailbreakingLLMInputOutputSafetyFilters/blob/main/Loss%20Achieved%20in%20less%20steps.png)

**Link do kodu:** https://colab.research.google.com/drive/1AFhgYBOAXzmn8BMcM7WUt-6BkOITstcn?pli=1#scrollTo=cNat4bxXVuH3&uniqifier=22

**Zestaw danych:** https://huggingface.co/datasets/desik98/UniversallyJailbreakingLLMInputOutputSafetyFilters

**Koszt**: Zapłaciłem **0 dolarów**. Biorąc pod uwagę, że mój zbiór danych liczy 200 milionów tokenów, kosztowałby mnie 1600 dolarów za epokę. Aby temu zapobiec, wykorzystałem 2 luki w systemie OpenAI. Udało mi się to znaleźć, biorąc pod uwagę, że przeprowadziłem wiele treningów z użyciem OpenAI w przeszłości. Oto te luki:
1. Jeśli mój trening kosztuje 100 dolarów, nie muszę płacić OpenAI 100 dolarów z góry. OpenAI zmniejsza kwotę do -100 po zakończeniu treningu.
2. Jeśli anuluję swoje zadanie w trakcie treningu, OpenAI nie pobiera żadnej opłaty.

W moim przypadku nie zapłaciłem żadnej kwoty OpenAI z góry, przesłałem zbiór danych o łącznej liczbie 200 milionów tokenów, a następnie anulowałem zadanie, gdy wiedziałem, że wartość funkcji straty osiągnęła odpowiednią wartość (0,3 w moim przypadku). Dzięki temu nie zapłaciłem nic OpenAI 🙂. Ale gdy faktycznie przeprowadzę testy wydajnościowe, nie będę mógł zatrzymać zadania w trakcie, i w takim przypadku będę musiał zapłacić OpenAI.

### Dlaczego publikuję teraz tę pracę, biorąc pod uwagę, że muszę jeszcze przeprowadzić testy wydajnościowe na ostatecznym modelu na zbiorze danych?
Na Uniwersytecie Kalifornijskim w Berkeley pojawiła się [niedawno praca (28 czerwca)](https://arxiv.org/pdf/2406.20053), która opiera się na podobnej intuicji, wykorzystując szyfry. Ale biorąc pod uwagę, że pracowałem ||'owo nad tym i technicznie uzyskałem wyniki (mniejsza wartość funkcji straty) nawet przed opublikowaniem tej pracy (21 czerwca). Dodatkowo zaproponowałem [ten pomysł 2 miesiące przed opublikowaniem tej pracy](https://www.apartresearch.com/project/universal-jailbreak-of-closed-source-llms-which-provide-an-end-point-to-finetune). Naprawdę myślałem, że nikt inny nie opublikuje czegoś podobnego, biorąc pod uwagę, że trzeba zrobić wiele rzeczy, takich jak oparte na intuicji podejście z użyciem szyfru, dodanie dużej ilości korpusów równoległych, podział tekstu na poziomie znaków itp. Ale biorąc pod uwagę, że ktoś inny opublikował pierwszy, chcę upewnić się, że prezentuję tutaj moje artefakty, aby ludzie uznali moją pracę za równoległą. Dodatkowo istnieją różnice w metodologii, o których wspomniałem poniżej. Uważam, że ta praca jest nowatorska, a artykuł został opracowany przez kilka osób jako zespół, a biorąc pod uwagę, że pracowałem nad tym sam, osiągnąłem podobne wyniki i chciałem je tutaj podzielić.

### Jakie są różnice między moim podejściem a opublikowanym artykułem?
1. Artykuł łamie model w 2 fazach. W pierwszej fazie uczą model języka szyfrów, a w drugiej fazie uczą go szkodliwych danych. Ja natomiast przeprowadziłem trening modelu w pojedynczej fazie, podając jednocześnie zaszyfrowany i szkodliwy zbiór danych. Problem z podejściem opisanym w artykule polega na tym, że po pierwszej fazie treningu OpenAI może użyć dostrojonego modelu do weryfikacji zbioru danych w drugiej fazie i może zasygnalizować, że zawiera on szkodliwe instrukcje. Może to się zdarzyć, ponieważ dostrojony model ma zrozumienie języka szyfrów.

2. Ja użyłem [Tokena Wyzwalacza](https://arxiv.org/abs/2401.05566), aby zwiększyć szkodliwość, czego nie robi artykuł.

3. Szyfr: Ja użyłem szyfru Cezara z przesunięciem 25, ponieważ Gpt4 go nie rozumie. Artykuł tworzy nowy szyfr podstawieniowy Walnut53, losowo permutując każdą literę za pomocą numpy.default_rng(seed=53).

4. Zadania treningowe - 

4.1 Moje zadania: Podaję równoległe korpusy z instrukcjami zawierającymi Szyfrowane Wejście - Szyfrowane Wyjście, Rozszyfrowane Wejście - Rozszyfrowane Wyjście, Rozszyfrowane Wejście - Szyfrowane Wyjście, Szyfrowane Wejście - Rozszyfrowane Wyjście, Szyfrowane Wejście - Szyfrowane Wyjście, gdzie najpierw dekoduję instrukcję, piszę odpowiedź jako zwykły tekst, a następnie koduję.

4.2 Zadania w artykule: Artykuł tworzy 4 różne zadania, wszystkie są Szyfr do Szyfru, ale różnią się strategią. 4 zadania to: Bezpośrednie Szyfrowane Wejście - Szyfrowane Wyjście, Szyfrowane Wejście - [Rozszyfrowane Wejście - Rozszyfrowane Wyjście - Szyfrowane Wyjście], Szyfrowane Wejście - [Rozszyfrowane Wyjście - Szyfrowane Wyjście], Szyfrowane Wejście - [Rozszyfrowane Wejście - Szyfrowane Wyjście]

5. Podstawowy zbiór danych do generowania instrukcji: Ja użyłem zbioru danych OpenOrca, a artykuł użył zbioru danych Alpaca.

6. Używam "kropek" między znakami dla lepszej tokenizacji, a artykuł używa "|".

7. Artykuł używa mniejszego zbioru danych składającego się z 20 000 instrukcji do nauczenia LLM nowego języka. Gratulacje dla nich za to.

### Inne podejścia, które próbowałem i które nie powiodły się, oraz jak poprawiłem swoje podejście:
Początkowo próbowałem użyć 12 000 instrukcji tłumaczenia Szyfr-NieSzyfr i 5 000 pytań, ale to nie przyniosło dobrego wyniku funkcji straty.

![Mniejsza wartość funkcji straty osiągnięta w mniejszej liczbie iteracji](https://github.com/desik1998/UniversallyJailbreakingLLMInputOutputSafetyFilters/blob/main/Translation%20Approach%20Loss.png?raw=true)

Następnie, po zapoznaniu się z literaturą dotyczącą nauczania nowych języków, dowiedziałem się, że podanie 70 000-100 000 instrukcji poprawia dokładność w zadaniach wtórnych. Postąpiłem zgodnie z tym podejściem i stworzyłem również korpusy równoległe, co pomogło w zmniejszeniu funkcji straty.
