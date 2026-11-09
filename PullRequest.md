XSS to jedna z popularnych podatności w aplikacjach webowych. Polega ona na tym, że atakujący znajduje lukę pozwalającą na wstrzyknięcie złośliwego kodu JavaScript do strony internetowej. Kiedy aplikacja wyświetli te sfałszowane dane bez uprzedniego zabezpieczenia, przeglądarka ofiary wykonuje ten kod, ufając, że pochodzi on z bezpiecznego źródła. W efekcie atakujący może uzyskać dostęp do np. ciasteczek sesyjnych użytkownika. Jedną z odmian tego ataku jest Persistent XSS. W przypadku tej formy ataku złośliwy skrypt przesyłany przez napastnika nie jednorazowo "odbija się" od przeglądarki, lecz zostaje trwale zapisany na serwerze - najczęściej w bazie danych np. w polu nazwy książki, opisie klienta czy treści komentarza na blogu. Oznacza to, że za każdym razem, gdy dowolny użytkownik odwiedzi podstronę wyświetlającą te dane, serwer pobierze zainfekowaną treść z bazy i wyśle ją w czystej postaci do przeglądarki. W aplikacji podatność XSS wystąpiła w zakładach Books, Customers i Loans. W każdej z tych zakładek przy dodawaniu nowego obiektu nie występowała weryfikacja poprawności przesyłanych danych. Payloadem do wykorzystania podatności może być np. `<img src=x onerror=alert(1)>`. 

Dowód wystąpienia podatności:

1. Customers

Uzupełniony fomularz:
<img width="945" height="893" alt="image" src="https://github.com/user-attachments/assets/011a455d-c61d-4d66-9704-1f3c2b8a7442" />

Wystąpienie XSS:

<img width="945" height="340" alt="image" src="https://github.com/user-attachments/assets/b2bb667c-d29c-4486-a2c8-32e5192db4bc" />

Problem dotyczył również opcji edycji tych pól.

Uzupełniony fomularz:
<img width="945" height="898" alt="image" src="https://github.com/user-attachments/assets/84e292e5-e04c-4a79-969b-e37cdb1be259" />

Wystąpienie XSS:

<img width="945" height="325" alt="image" src="https://github.com/user-attachments/assets/e7a5c617-94fa-40d0-be59-9fdaa322fab9" />


2. Customers

Uzupełniony fomularz:

<img width="945" height="726" alt="image" src="https://github.com/user-attachments/assets/6fe24e17-3d1e-4c14-9297-40698929e5bc" />


Wystąpienie XSS:

<img width="945" height="321" alt="image" src="https://github.com/user-attachments/assets/c5cc6f8f-7ecd-4d6d-b618-2b6bf4268862" />

Tu problem też dotyczył edycji.

Uzupełniony fomularz:

<img width="945" height="718" alt="image" src="https://github.com/user-attachments/assets/aa442d74-56c4-46b6-bfba-43e5a0659cdd" />


Wystąpienie XSS:

<img width="945" height="373" alt="image" src="https://github.com/user-attachments/assets/edef42ff-40a3-4207-9ed0-5b2a7a9290c9" />

3. Loans

Uzupełniony fomularz:

<img width="945" height="916" alt="image" src="https://github.com/user-attachments/assets/977699bc-6584-4a79-96e2-351032d03d7f" />

Wystąpienie XSS:

<img width="945" height="287" alt="image" src="https://github.com/user-attachments/assets/2549eb1a-125c-4b02-b8b7-5312fd3aab92" />



W celu poprawy bezpieczeństwa aplikacji oraz ochrony przed wstrzykiwaniem złośliwego kodu, dodano funkcję sanitize_input oraz wyrażenie regularne contains_html_tag. W celu poprawy walidacji danych wejściowych podczas tworzenia nowej pozycji, w /create zmodyfikowano sposób przetwarzania żądania JSON. Teraz pobrane pola takie jak nazwa, autor czy typ książki są najpierw przepuszczane przez wspomnianą funkcję czyszczącą, a system dodatkowo upewnia się, że po usunięciu ewentualnych tagów pola te nie pozostały puste - w przeciwnym razie od razu zwracany jest błąd z kodem 400. Z kolei w celu poprawy kontroli modyfikowanymi danymi podczas edycji, wprowadzono mechanizm sprawdzający za pomocą contains_html_tag.search(), czy nowe imię lub autor nie zawierają niedozwolonych znaczników HTML. W celu poprawy stabilności aktualizacji samego rekordu, przypisywanie wartości do obiektu książki zostało opatrzone dodatkowymi warunkami sprawdzającymi, czy dane pole faktycznie zostało przesłane w żądaniu.

Podobne zmiany wprowadzono dla /customers. W celu poprawy walidacji danych wejściowych podczas tworzenia nowego klienta, w /create zmodyfikowano sposób obsługi przesłanych danych z formularza. Teraz pobrane pola takie jak nazwa oraz miasto są najpierw przepuszczane przez funkcję czyszczącą, a system dodatkowo upewnia się, że po usunięciu ewentualnych tagów pola te nie pozostały puste - w przeciwnym razie od razu zwracany jest błąd z kodem 400 i informacją o nieprawidłowych danych. Z kolei w celu poprawy kontroli nad modyfikacją danych klienta, wprowadzono mechanizm sprawdzający za pomocą contains_html_tag.search(), czy nowa nazwa lub miasto nie zawierają niedozwolonych znaczników HTML. W przypadku wykrycia takich tagów, edycja zostaje zablokowana z komunikatem o błędzie.

W zakładce Loans wprowadzono podobne modyfiakcje. W celu poprawy walidacji i oczyszczania danych wejściowych podczas rejestrowania wypożyczenia, w /create zmieniono sposób pobierania danych z formularza. Teraz pola przechowujące nazwę klienta oraz tytuł książki są najpierw przepuszczane przez funkcję czyszczącą sanitize_input, co pozwala skutecznie usunąć wszelkie niepożądane tagi HTML zanim dane zostaną użyte do wyszukania książki w bazie lub zapisane jako nowe wypożyczenie.

W projekcie dodano również folder tests, który zawiera testy dla tworzenia i edycji books i customers oraz tworzenia loans.
