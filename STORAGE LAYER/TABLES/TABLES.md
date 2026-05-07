[[STORAGE LAYER]]

kiedy okreslac dlugosc znakow w kolumnie (np VARCHAR(50)):
nie ma to wplywu na query performance pod wzgledem efektywnosci, bo snowflake i tak kompresuje kolumny. Ale jak wiemy ze nasze wpisy maja przewidywalna ilosc znakow to mozna ustawic sobie odpowiedni varchar, zeby potem wykrywac latwiej bledy/abnormalities - jak np do varchar(10) pojdzie 50 znakow, to bedzie blad.


