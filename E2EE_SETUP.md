# Llogaritë e Mia — përgatitja e enkriptimit E2EE

## Gjendja e këtij ndërtimi

Ky ndërtim dhe udhëzues janë testuar me të dhëna të simuluara në shfletues dhe me Firebase Firestore Emulator; testet nuk u lidhën me Firebase-in e prodhimit dhe nuk lexuan, kopjuan apo migruan të dhëna reale. Publikimi i rregullave në Firebase bëhet veçmas dhe nuk aktivizon vetvetiu E2EE; enkriptimi fillon vetëm kur aplikacioni krijon kopjen rezervë dhe kryen migrimin.

**Modeli i çelësit i zgjedhur:** fjalëkalim privat i kasafortës (minimumi 16 karaktere), i veçantë nga Firebase, plus një kod rikuperimi njëpërdorimësh 20-byte. Rivendosja e fjalëkalimit Firebase nuk e ndryshon çelësin e kasafortës.

**Kujtesa e pajisjes:** pas zhbllokimit të parë, aplikacioni ruan në IndexedDB të këtij profili shfletuesi një çelës Web Crypto jo të eksportueshëm. Kjo nuk ruan fjalëkalimin dhe nuk sinkronizohet në pajisje të tjera; pas hyrjes së zakonshme në Firebase, kjo pajisje mund ta hapë kasafortën pa pyetur përsëri. “Dil” ruan besimin lokal. Nëse dëshiron që pajisja të kërkojë sërish fjalëkalimin, shko te **Cilësimet → Llogaria → Hiq çelësin e ruajtur**. Në pajisje/shfletues të ri, ose pas pastrimit të të dhënave të shfletuesit, duhet fjalëkalimi privat ose kodi i rikuperimit. Mos e përdor këtë lehtësi në pajisje të përbashkët: kush ka qasje në të njëjtin profil të shfletuesit dhe llogarinë Firebase mund t'i hapë të dhënat.

## Para se të nisësh

1. Ruaj një kopje të rregullave ekzistuese Firestore. Skedari `firestore.rules` dhe seksioni “Siguria e Firestore” brenda aplikacionit përmbajnë rregulla që lejojnë leximin e të dhënave të vjetra vetëm nga pronari, por pranojnë shkrime të reja vetëm si zarfe të enkriptuara.
2. Kontrollo dhe bashko rregullat me çdo leje tjetër që përdor projekti. Mos i zëvendëso rregullat ekzistuese pa e verifikuar ndikimin. Publiko rregullat përpara se të krijosh kopjen rezervë ose të nisësh migrimin. Ato u testuan në emulator; kontrollo në Firebase Console/CLI që rregullat e publikuara janë pikërisht ato që ke shqyrtuar. Ngarkimi i `firestore.rules` në GitHub nuk i publikon.
3. Mbylli të gjitha skedat dhe kopjet e vjetra të aplikacionit në çdo pajisje. Pas publikimit të rregullave, një kopje e vjetër duhet të mos jetë në gjendje të shkruajë tekst të thjeshtë.
4. Përdor versionin e ri në Safari/Chrome të përditësuar dhe në HTTPS. E2EE përdor Web Crypto; skedari lokal mund të mos ketë një kontekst të sigurt në çdo shfletues.

## Rrjedha e kopjes dhe migrimit

1. Hyr me llogarinë Firebase.
2. Krijo kasafortën dhe ruaj fjalëkalimin privat veçmas nga fjalëkalimi i hyrjes.
3. Konfirmo se mbylle versionet e vjetra dhe publikove rregullat. Aplikacioni pastaj lexon nga serveri dokumentin `users/{UID}` dhe shtatë koleksionet `clients`, `debts`, `debtHistory`, `rent`, `expenses`, `noteFolders`, `notes`.
4. Shkarko skedarin JSON **para** enkriptimit. Ai përmban të dhëna në tekst të thjeshtë, ID-të e dokumenteve, UID-në dhe SHA-256 për kontrollin e dëmtimit/ndryshimit aksidental. Hape dhe kontrollo skedarin para se të vazhdosh; ruaje jashtë telefonit në një vend të sigurt. Mos e dërgo me email ose në një shërbim cloud të pasigurt.
5. Ruaj veçmas kodin e rikuperimit dhe konfirmo kutitë përkatëse. Kodi shfaqet vetëm gjatë konfigurimit/rikuperimit. Humbja e fjalëkalimit privat dhe e kodit mund të bllokojë qasjen te rekordet e enkriptuara.
6. Zgjidh vazhdimin. Migrimi përpunon të dhënat në faqe, kapërcen rekordet që janë enkriptuar tashmë, pastaj kontrollon sërish root-in dhe të gjitha koleksionet. Nëse ndërpritet, mos përdor versionin e vjetër: hape versionin e ri, zhblloko kasafortën dhe provo sërish.

**Nëse humbet kodin e rikuperimit, por ke fjalëkalimin privat:** kodi i vjetër nuk mund të lexohet nga Firebase, sepse ruhet vetëm mbështjellësi i enkriptuar. Hape kasafortën, shko te **Cilësimet → Kodi i rikuperimit**, shkruaj fjalëkalimin privat dhe krijo kod të ri. Ruaje kodin e ri jashtë aplikacionit dhe konfirmoje para se të vazhdosh. Vetëm pas konfirmimit zëvendësohet mbështjellësi i kodit të vjetër; shënimet dhe çelësi që i enkripton nuk ndryshohen.

Aplikacioni ka edhe veprimin **“Rikthe nga kopja JSON”** për importimin e një kopjeje të së njëjtës llogari. Ai zëvendëson rekordet aktuale, prandaj kërkon konfirmim dhe enkripton të dhënat përsëri me çelësin e kasafortës që është hapur. Kopja e importuar nuk dërgohet si tekst i thjeshtë te Firestore.

## Çfarë mbron dhe çfarë nuk mbron

- Përmbajtja e rekordit kodohet në pajisje me AES-GCM; çelësi i të dhënave mbështillet me PBKDF2-SHA-256 dhe me çelësin e kodit të rikuperimit.
- Emrat e koleksioneve, ID-të e dokumenteve, UID-ja, koha dhe madhësia e përafërt e tekstit të enkriptuar mbeten metadata të dukshme për Firebase.
- E2EE nuk mbron pajisjen ose shfletuesin kur kasaforta është e hapur ose kur çelësi mbahet mend lokalisht, pajisjen e komprometuar, kopjet plaintext që ti ruan vetë, skriptet me të njëjtin burim që ekzekutohen në faqe, apo një klient të vjetër që vazhdon të punojë pa rregulla të publikuara.
- Rregullat Firestore nuk mund të ndryshohen nga ky skedar HTML. Ato duhen publikuar nga pronari i projektit.
- Rivendosja e fjalëkalimit Firebase nuk e ndryshon fjalëkalimin e kasafortës.
