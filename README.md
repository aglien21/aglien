# GitHub Pages build — Llogaritë e Mia v11

## Publikimi i faqes

Kopjo skedarët e këtij direktorie në rrënjën e repository-t, jo brenda një direktorie tjetër. `index.html` duhet të jetë në të njëjtin nivel me `manifest.json` dhe ikonat. Te GitHub, hap **Settings → Pages** dhe zgjidh branch-in kryesor dhe **/(root)**.

Në Firebase Authentication, shto domenin e faqes GitHub Pages (p.sh. `emriyt.github.io`) te **Authorized domains** nëse nuk është tashmë i shtuar. Përdor faqen HTTPS.

## Firestore

`firestore.rules` është kopje për kontroll dhe versionim; GitHub Pages nuk i publikon rregullat. Publikoji veçmas në Firebase Console ose me Firebase CLI dhe verifiko që janë rregullat e duhura.

## Kujdes me skedarët privatë

Mos ngarko kopje rezervë JSON, kode rikuperimi, fjalëkalime ose çelësa privatë të Firebase Admin. Pas zhbllokimit të parë, pajisja mund ta kujtojë çelësin lokalisht; në pajisje të përbashkët përdor **Cilësimet → Llogaria → Hiq çelësin e ruajtur** para se ta lësh pa mbikëqyrje.
