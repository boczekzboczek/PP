#include <iostream>
#include <Windows.h>
#include <fstream>
#include <sstream>
 
using namespace std;
 
struct Date
{
    string dzien, miesiac, rok;
 
    string GetDate()
    {
        return dzien  + "-" + miesiac + "-" + rok;
    }
};
 
struct Osoba
{
    int    ID;
    string imie;
    string nazwisko;
    string miasto;
    Date   dataUrodzenia;
 
    void WypiszDane()
    {
        cout << "ID: " << ID << ", Imie: " << imie << ", Nazwisko: " << nazwisko << ", Miasto: " << miasto << ", Data urodzenia: " + dataUrodzenia.GetDate() << endl;
    }
};
 
struct Lista
{
    struct Ogniwo
    {
        Ogniwo * poprzedni;
        Ogniwo * nastepny;
        Osoba osoba;
    };
 
    struct Iterator
    {
        Ogniwo * wsk;
        Iterator next(Iterator it)
        {
            it.wsk = it.wsk->nastepny;
            return it;
        }
    };
 
    Iterator poczatek;
    Iterator koniec;
    int size;
 
    Lista(Osoba osoba)
    {
        Ogniwo * o = new Ogniwo(); //tworzymy nowy wskaŸnik typu Osoba
        o->nastepny = o;           //jest tylko jeden segment wiêc o bêdzie wskazywaæ na samego siebie
        o->poprzedni = o;
        o->osoba = osoba;
        poczatek.wsk = o;          //ustalamy poczatkowy wskaznik
        koniec.wsk = o;            //ustalamy koncowy wskaznik
        size = 1;
    }
 
    Iterator Insert(Iterator it, Osoba osoba)
    {
        Ogniwo * a = it.wsk;
        Ogniwo * b = new Ogniwo();
        Ogniwo * c = a->nastepny;
        b->nastepny = c;
        b->poprzedni = a;
        b->osoba = osoba;
        c->poprzedni = b;
        a->nastepny = b;
        if (b->nastepny == b)
            koniec.wsk = b;
        size++;
        return it;
    }
 
    Iterator Delete(Iterator it)
    {
        Ogniwo * b = it.wsk;
        Ogniwo * a = b->poprzedni;
        Ogniwo * c = b->nastepny;
        a->nastepny = c;
        c->poprzedni = a;
        if (poczatek.wsk == b)
            poczatek.wsk = c;   //jeœli usuwamy pierwszy segment listy, musimy ustaliæ nowy
        if (koniec.wsk == b)
            koniec.wsk = a;     //jeœli usuwamy ostatni segment listy, musimy ustaliæ nowy
        delete b;
        it.wsk = a;
        size--;
        return it;
    }
 
    Iterator Get(Osoba osoba)
    {
        Iterator it = poczatek;
        for (int i = 0; i < size; i++)
        {
            Osoba x = it.wsk->osoba;
            if (&x == &osoba) break;
            else it.wsk = it.wsk->nastepny;
        }
        return it;
    }
};
 
Lista lista = Lista(Osoba());
bool zainicjalizowanoListe = false;
 
void Error()
{
    cin.ignore();
    cin.clear();
    cout << "Podano zly typ liczby.\n";
    system("pause > nul");
}
 
void DodajOsobe(Lista::Iterator it = lista.poczatek, bool clear = true)
{
    if (clear)
        system("cls");
    Osoba o;
    o.ID = !zainicjalizowanoListe ? 1 : lista.size + 1;
 
    cout << "Podaj imie\n";
    cin >> o.imie;
    cout << "Podaj nazwisko\n";
    cin >> o.nazwisko;
    cout << "Podaj miasto\n";
    cin >> o.miasto;
 
    cout << "Podaj dzien urodzin\n";
    cin >> o.dataUrodzenia.dzien;
    cout << "Podaj miesiac urodzin\n";
    cin >> o.dataUrodzenia.miesiac;
    cout << "Podaj rok w ktorym osoba sie urodzila\n";
    cin >> o.dataUrodzenia.rok;
 
    if (!cin)
    {
        system("cls");
        Error();
        return;
    }
 
    if (!zainicjalizowanoListe)
    {
        lista = Lista(o);
        zainicjalizowanoListe = true;
    }
    else
        lista.Insert(it, o);
}
 
void UsunOsobe()
{
    string nazwisko;
    bool znaleziono = false;
 
    cout << "Podaj Nazwisko osoby ktora chcesz usunac z listy\n";
    cin >> nazwisko;
 
    Lista::Iterator it = lista.poczatek;
    for (int i = 0; i < lista.size; i++)
    {
        Osoba osoba = it.wsk->osoba;
        if (nazwisko == osoba.nazwisko)
        {
            lista.Delete(it);
            znaleziono = true;
            break;
        }
        else
            it.wsk = it.wsk->nastepny;
    }
    if (!znaleziono)
        cout << "Nie znaleziono osoby w bazie.\n";
    system("pause>nul");
}
 
void WyszukajOsobe()
{
    string nazwisko;
    bool znaleziono = false;
 
    cout << "Podaj Nazwisko osoby ktora chcesz znalezc\n";
    cin >> nazwisko;
 
    Lista::Iterator it = lista.poczatek;
    for (int i = 0; i < lista.size; i++)
    {
        Osoba osoba = it.wsk->osoba;
        if (nazwisko == osoba.nazwisko)
        {
            osoba.WypiszDane();
            znaleziono = true;
        }
        it.wsk = it.wsk->nastepny;
    }
    char wybor = 'T';
    if (znaleziono)
        system("pause>nul");
    else
    {
        cout << "Nie znaleziono osoby o danym nazwisku, powrocic do menu?(T/N)\n";
        cin >> wybor;
    }
    if (wybor != 'T' && wybor != 't')
        WyszukajOsobe();
}
 
void WyswietlWgNazwiska()
{
    if (zainicjalizowanoListe)
    {
        Lista::Iterator it = lista.koniec;
        for (int i = 0; i < lista.size; i++)
        {
            it.wsk->osoba.WypiszDane();
            it.wsk = it.wsk->poprzedni;
        }
    }
    else
        cout << "Lista jest pusta.\n";
    system("pause>nul");
}
 
void ModyfikujOsobe()
{
    int ID;
    bool znaleziono = false;
    cout << "Podaj ID osoby ktora chcesz zmodyfikowac\n";
    cin >> ID;
 
    if (!cin)
    {
        system("cls");
        Error();
        return;
    }
 
    Lista::Iterator it = lista.poczatek;
    for (int i = 0; i < lista.size; i++)
    {
        Osoba osoba = it.wsk->osoba;
        if (ID == osoba.ID)
        {
            osoba.WypiszDane();
            znaleziono = true;
            DodajOsobe(it, false);
            lista.Delete(it);
            break;
        }
        else
            it.wsk = it.wsk->nastepny;
    }
    char wybor = 'T';
    if(!znaleziono)
    {
        cout << "Nie znaleziono osoby o danym ID, powrocic do menu?(T/N)\n";
        cin >> wybor;
    }
 
    if (wybor != 'T' && wybor != 't')
        ModyfikujOsobe();
}
 
void ZapiszDoPliku()
{
    ofstream myFile;
    string nazwaPliku;
    cout << "Podaj nazwe pliku do ktorego chcesz zapisac dane.\n";
    cin >> nazwaPliku;
 
    myFile.open((nazwaPliku + ".txt").c_str());
    Lista::Iterator it = lista.poczatek;
    for (int i = 0; i < lista.size; i++)
    {
        Osoba osoba = it.wsk->osoba;
        myFile << osoba.ID << " " << osoba.imie << " " << osoba.nazwisko << " " << osoba.miasto << " " << osoba.dataUrodzenia.dzien << " " << osoba.dataUrodzenia.miesiac << " " << osoba.dataUrodzenia.rok << " \n";
        it.wsk = it.wsk->nastepny;
    }
    myFile.close();
}
 
void OdczytajZPliku()
{
    string nazwaPliku;
    cout << "Podaj nazwe pliku z ktorego chcesz odczytac dane.\n";
    cin >> nazwaPliku;
 
    string line;
    ifstream myFile((nazwaPliku + ".txt").c_str());
    if (myFile.is_open())
    {
        zainicjalizowanoListe = false;
        while (getline(myFile, line))
        {
            Osoba o;
            string elementOsoby = ""; //przechowuje pojedynczy element osoby
            int counter = 0;          //okreœla który element osoby aktualnie ustalamy
 
            for (int i = 0; i < line.length(); i++)
            {
                if (line[i] == ' ')
                {
                    if (counter == 0) o.ID = atoi(elementOsoby.c_str());
                    else if (counter == 1) o.imie = elementOsoby;
                    else if (counter == 2) o.nazwisko = elementOsoby;
                    else if (counter == 3) o.miasto = elementOsoby;
                    else if (counter == 4) o.dataUrodzenia.dzien = elementOsoby.c_str();
                    else if (counter == 5) o.dataUrodzenia.miesiac = elementOsoby.c_str();
                    else o.dataUrodzenia.rok = elementOsoby.c_str();
                    elementOsoby = "";
                    counter++;
                    continue;
                }
                elementOsoby += line[i];
            }
            if (!zainicjalizowanoListe)
            {
                lista = Lista(o);
                zainicjalizowanoListe = true;
            }
            else
                lista.Insert(lista.poczatek, o);
        }
    }
    else
    {
        cout << "Plik nie zostal odnaleziony.\n";
        system("pause>nul");
    }
    myFile.close();
}
 
int main()
{
    char wybor = '0';
    while (wybor != '8')
    {
        cout << "\tMENU\n";
        cout << "1. Dodaj nowa osobe\n";
        cout << "2. Usun nowa osobe\n";
        cout << "3. Wyszukaj osobe\n";
        cout << "4. Wyswietl baze osob wg nazwisk\n";
        cout << "5. Modyfikuj osobe po ID\n";
        cout << "6. Zapisz do pliku\n";
        cout << "7. Odczytaj z pliku\n";
        cout << "8. Koniec pracy\n";
        cin >> wybor;
        system("cls");
        switch (wybor){
        case '1':DodajOsobe();         break;
        case '2':UsunOsobe();          break;
        case '3':WyszukajOsobe();      break;
        case '4':WyswietlWgNazwiska(); break;
        case '5':ModyfikujOsobe();     break;
        case '6':ZapiszDoPliku();      break;
        case '7':OdczytajZPliku();     break;
        }
        system("cls");
    }
    return 0;
}
