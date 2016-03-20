# Vilniaus miesto problemų web serviso aprašymas

Turinys

1. Miesto problemų informacinės sistemos plėtinio (toliau - MPISP) duomenys
2. Web servisų aprašymas
	1. Metodas Login
	2. Metodas Logout
	3. Metodas Register
	4. Metodas GetProblemTypes
	5. Metodas NewProblem
	6. Metodas GetProblems
	7. Metodas GetProblem
	8. Metodas GetAddresses


## Miesto problemų informacinės sistemos plėtinio (toliau - MPISP) duomenys

| MPISP lauko pavadinimas | Lauko tipas |
|:---|:---|
| Problemos aprašymas | Tekstas |
| Problemos tipas | Pasirenkamas laukas (administruojamas) |
| Problemos statusas | Pasirenkamas laukas (fiksuotas) |
| Adresas | Imamas iš žemėlapio (įvedamas) |
| X koordinatė | Nurodomas žemėlapyje (įvedamas) |
| Y koordinatė | Nurodomas žemėlapyje (įvedamas) |
| El. paštas | Tekstas |
| Telefonas | Tekstas |
| Pranešimo aprašymas | Tekstas |

Galimi problemos tipai:

*	Eismo organizavimas
*	Energetika ir inžineriniai tinklai
*	Gatvių apšvietimas
*	Gatvių priežiūra ir tvarkymas
*	Gyvūnų laikymo Vilniaus miesto Savivaldybės teritorijoje taisyklių pažeidimai
*	Išorinės reklamos įrengimo taisyklių pažeidimai
*	Kapinių priežiūra
*	Komunalinių atliekų tvarkymas
*	Pastatų administravimas
*	Tiltų, viadukų, tunelių ir estakadų priežiūra
*	Triukšmo prevencijos viešosiose vietose taisyklių pažeidimas
*	Tvarkymo ir švaros taisyklių pažeidimas
*	Viešasis transportas
*	Vilniaus miesto aplinkos tvarkymas
*	Stovėjimo tvarkos gyvenamosiose zonose ir kiemuose pažeidimai
*	Statinių priežiūra

Galimi problemos statusai:

*	Registruota
*	Atlikta

X ir Y koordinatės pateikiamos WGS-84 standartu.

## Web servisų aprašymas

Duomenų apsikeitimui tarp klientinės programos ir serverio bus naudojami JSON-RPC pagrindu veikiantys servisai. Visos tokių web servisų užklausos turi bendrąjį pavidalą:
```
{ 
	"method" : metodo_pavadinimas, 
	"params": [ parametrai ], 
	"id": parinktas_id 
}
```
Visi serverio atsakymai įgyja pavidalą:
```
{ 
	"result" : rezultatas, 
	"error": klaida arba null, 
	"id": užklausoje_naudotas_id 
} 
```
Savivaldybės problemų fiksavimo sistemoje realizuotas web servisas pateikia tokius metodus:

* Login – vykdo naudotojo prisijungimą prie sistemos
* Logout – atjungia naudotoją nuo sistemos
* Register – užregistruoja naują naudotoją sistemoje
* GetProblemTypes – grąžina galimų problemos tipų sąrašą
* NewProblem – registruoja naują problemą
* GetProblems – grąžina problemų, atitinkančių filtravimo kriterijus, sąrašą
* GetProblem – pagal problemos kodą grąžina problemos duomenis
* GetAddresses – gražina galimus adresus pagal pateiktus parametrus

### Metodas Login

Pagrindinė šio metodo paskirtis - naudotojo autentifikavimas sistemoje. Metodui pateikiami du parametrai – naudotojo vardas ir slaptažodis užkoduotas viešuoju raktu naudojant „Certificate-based encryption“ šifravimą bei base64. Pagrindinis metodo rezultatas – sesijos id, kuris vėliau naudojamas kitų metodų kvietime.

Užklausos pavyzdys:
```
{ 
	"method": "login", 
	"params": [ 
		{ 
			"login": "test", 
			"password": "5AF12FFDEA125"
		}
	],
	"id": 1
}
```

Serviso atsakymo pavyzdys:
```
{ 
	"result": {
		"session_id": "51AEF587DAE" 
	} 
	"error": null, 
	"id": 1
}
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |
| 3 | Nenurodyti būtini parametrai |
| 4 | Klaidingas vartotojo vardas arba slaptažodis |

###	Metodas Logout

Šio metodo paskirtis – įvykdyti naudoto atsijungimo nuo sistemos veiksmą. Vienintelis perduodamas parametras – sesijos id. Rezultato laukas užpildomas taip/ne reikšme.

Užklausos pavyzdys:
```
{ 
	"method": "logout", 
	"params": [ 
		{ 
			"session_id": "51AEF587DAE" 
		}
	],
	"id": 2
}
```
Serviso atsakymo pavyzdys:
```
{ 
	"result": true,
	"error": null, 
	"id": 2
}
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |
| 3 | Nenurodyti būtini parametrai |
| 5 | Neteisingas sesijos id |
	
### Metodas Register

Metodo paskirtis – sukurti naujas naudotojų paskyras sistemoje. Metodui kaip parametrai pateikiami naudotojo registravimosi duomenys (naudotojo vardas, slaptažodis, elektroninis paštas, telefono numeris, telefono IMEI). Po sėkmingos registracijos naudotojas automatiškai autentifikuojamas sistemoje, todėl sėkmingos registracijos metu kaip rezultatas turėtų būti grąžinamas sesijos id, kitu atveju grąžinamas klaidos kodas.

Užklausos pavyzdys:
```
{ 
	"method": "register", 
	"params": [ 
		{ 
			"username": "test",
			"password": "test",
			"email": "test@test.tt",
			"phone_no": "+37066666666",
			"imei": "1EA11122234",
		}
	],
	"id": 3
}
```

`email`, `phone_no` ir `imei` nėra privalomi laukai. Jei jie nėra pateikiami, tai vietoje jų turi būti nurodoma `null`.

Serviso atsakymo pavyzdys:
```
{ 
	"result": {
		"session_id": "51AEF587DAE" 
	} 
	"error": null, 
	"id": 3
}
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |
| 3 | Nenurodyti būtini parametrai |
| 6 | Toks naudotojas jau yra |
| 7 | Toks elektroninis paštas jau užregistruotas |
| 8 | Toks telefono numeris jau užregistruotas |
| 9 | Toks telefono IMEI jau užregistruotas |

### Metodas GetProblemTypes

Problemos tipo laukas yra laisvai konfigūruojamas, todėl klientinei programai reikalingas metodas, kuris nurodytų, kokie problemų tipai šiuo metu galimi. Tokia ir yra metodo GetProblemTypes paskirtis. Metodas nereikalauja jokių parametrų, o kaip rezultatą grąžina tipų sąrašą masyvo pavidalu.

Užklausos pavyzdys:
```
{ 
	"method": "get_problem_types", 
	"params": null,
	"id": 4
}
```

Serviso atsakymo pavyzdys:
```
{ 
	"result": [ "Tipas1", "Tipas2", "Tipas3" ] 
	"error": null, 
	"id": 4
}
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |

### Metodas NewProblem
Šio metodo paskirtis – įvesti naują problemą problemų fiksavimo sistemoje. Kaip pradiniai parametrai pateikiama problemą aprašanti informacija (problemos aprašymas, tipas, adresas, x koordinatė, y koordinatė, el. paštas, telefonas, pranešimo aprašymas, problemą patikslinanti nuotrauka). Adreso reikšmė užpildoma pasinaudojus „GetAddresses“ metodu. Nuotrauka užklausoje koduojama Base64 formatu. Nuotrauka nėra būtinas parametras ir jei jos nėra, tai photo lauke turi būti nurodomas `null`. Yra galimybė prikabinti daugiau negu vieną nuotrauką. Tokiu atveju nuotraukos parametre nurodomas nuotraukų masyvas. Taip pat kaip parametras būtinai turi būti nurodomas sesijos id. Kaip rezultatas grąžinamas požymis taip/ne.

Užklausos pavyzdys:
```
{ 
	"method": "new_problem", 
	"params": {
		"session_id": "51AEF587DAE",
		"description": "test",
		"type": "Tipas1",
		"address": "Laisves pr. 2",
		"x": 10.1518,
		"y": 2.01118,
		["email": "test@test.lt",]
		["phone": "+37066666666",]
		["message_description": "test2",]
		["photo": "TWFuIGlzIGRpc…."]
	},
	"id": 5
}
```

`email`, `phone`, `photo` ir `message_description` nėra privalomi laukai. Jei jie nėra pateikiami, tai vietoje jų turi būti nurodoma `null`.

Serviso atsakymo pavyzdys:
```
{ 
	"result": true 
	"error": null, 
	"id": 5
}
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |
| 3 | Nenurodyti būtini parametrai |
| 5 | Neteisingas sesijos id |

### Metodas GetProblems

Metodo paskirtis – pateikti problemų sąrašą, atitinkanti užklausos filtrų parametrus. Užklausoje kaip parametrai pateikiami įvairūs filtrai (tipo filtras, aprašymo dalies filtras, būsenos filtras, įvedusio asmens filtras, įvedimo datos filtras, gatvės filtras) taip pat nurodomas įrašų gražinimo kiekis su įrašo numeriu, nuo kurio pradėti. Nė vienas iš parametrų nėra būtinas ir jei kurio nors nurodyti nepageidaujama, tai vietoje jo nurodoma `null`. Metodo rezultatas – kriterijus atitinkančių problemų sąrašas.

Užklausos pavyzdys:
```
{ 
	"method": "get_problems", 
	"params": {
		"description_filter": null,
		"type_filter": "Tipas1",
		"address_filter": null,
		"reporter_filter": null,
		"date_filter": null,
		"status_filter": null,
		"start": null,
		"limit": 2
	},
	"id": 6
}
```

Serviso atsakymo pavyzdys:
```
{ 
	"result": [
		{
			"docNo": "test"
			"description": "test",,
			"status": "Atlikta",
			"address": "Laisves pr. 2",
			"answer": "Problema buvo išspresta",
			"from_app": false,
			"x": 10.1518,
			"y": 2.01118,
		},
		{
			"docNo": "test",
			"description": "test2",
			"status": "Vykdoma",
			"address": "Laisves pr. 2",
			"answer": null,
			"from_app": true,
			"x": 10.1518,
			"y": 2.01118,
		}
	],
	"error": null, 
	"id": 6
}
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |

### Metodas GetProblem

Metodo paskirtis – pagal užklausos id parametrą pateikti išsamius problemos duomenis.

Užklausos pavyzdys:
```
{ 
	"method": "get_problem", 
	"params": {
		 "id": "E50-1098-(3.2.8E-UK7)"
	},
	"id": 7
}
```

Serviso atsakymo pavyzdys:
```
{
	"id":"7",
	"result": [
		{
			"docNo":"E50-1098-(3.2.8E-UK7)",
			"description":"Turime problemų, nes Ratnyčios g. yra neužtaisytų duobių.",
			"entry_date":"2012-11-26 09:56:02",
			"reporter":"Testas Testinis",
			"assignee":"Statinių skyrius",
			"status":"Registruota",
			"type":"Gatvių priežiūra ir tvarkymas",
			"address":"Ratnyčios g. 9",
			"answer":null,
			"from_app":false,
			"x":25.279740621815,
			"y":54.706908316588,
			"email":"testas@testas.lt",
			"phone":" 868585858",
			"message_description":"test2",
			"photo":null
		}
	],
	"error":null
}
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |

### Metodas GetAddresses

Metodo paskirtis – pagal pateiktą tekstą gražinti galimus adresus.

Užklausos pavyzdys:
```
{ 
	"method": "get_addresses", 
	"params": {
		"string": "Mint"
		"limit": 2
	},
	"id": 8
}
```

Serviso atsakymo pavyzdys (ne JSON-RPC pavidalo atsakymas):
```
[
	{
		"id":"25439",
		"value":"Minties g. 1"
	},
	{
		"id":"3728",
		"value":"Minties g. 10"
	}
]
```

Galimi klaidos kodai:

| Kodas | Aprašymas |
|:---|:---|
| 1 | Serverio klaida |
| 2 | Neteisingai suformuota užklausa |
