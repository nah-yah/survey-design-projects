# Conception d'un formulaire XLSForm - Économie informelle urbaine, Haïti
# Konsepsyon yon XLSForm - Ekonomi enfòmèl iben, Ayiti

**Outils / Zouti :** ONA, KoBoToolbox, XLSForm, Enketo  
**Période / Peryòd :** Novembre - Décembre 2025 / Novanm - Desanm 2025  
**Cible / Sib :** 1 200 répondants / 1 200 moun k ap reponn  
**Commanditaire fictif / Kòmandatè fiksyon :** Fondation Atlas pour le Développement (FAD)  
**Statut / Estati :** Projet de portfolio (instrument fictif) / Pwojè pòtfolyo (enstriman fiksyon)

---

## Contexte

Le Département de l'Ouest concentre la part la plus dense de l'activité économique informelle haïtienne. Ses douze communes, de Port-au-Prince à Léogâne en passant par Croix-des-Bouquets, abritent des centaines de milliers d'opérateurs sans registre formel : marchands ambulants, chauffeurs de moto-taxi, couturières, réparateurs d'électronique, cuisinières. Ces travailleurs ne figurent dans aucune base de données institutionnelle et restent systématiquement hors du périmètre des enquêtes entreprises standard.

Ce projet conçoit un instrument de collecte de données terrain destiné à documenter leurs conditions économiques réelles : revenus, charges, accès au financement (bancaire, mobile money, sol et sabotaj), exposition aux chocs, et besoins de développement. La cible est fixée à 1 200 répondants, à raison de 100 par commune, avec surreprésentation des zones à forte densité d'activité informelle (Port-au-Prince, Delmas, Pétion-Ville, Carrefour, Cité Soleil).

Le formulaire est bilingue français-créole haïtien pour répondre aux conditions réelles du terrain, où les enquêteurs passent d'une langue à l'autre selon le profil du répondant.

---

## Architecture du formulaire

Le fichier XLSForm contient 92 lignes dans l'onglet `survey`, 155 options dans l'onglet `choices` réparties sur 21 listes distinctes, et un onglet `settings` configuré pour ONA.

| Section | Contenu |
|---|---|
| A | Identification enquêteur, sélection de commune, capture GPS, code auto-généré |
| B | Profil répondant : sexe, âge, instruction, situation matrimoniale, taille du ménage, personnes à charge |
| C | Activité principale : secteur, sous-secteur (filtré), lieu, statut formel, ancienneté, employés |
| D | Revenus et charges : revenu mensuel estimé, charges, marge brute (calculée), baisse des revenus |
| E | Accès au financement : compte bancaire, mobile money, sol/sabotaj, dettes et ratio dette/revenu |
| F | Activités secondaires : groupe répété, jusqu'à 3 itérations |
| G | Impact des chocs : insécurité, catastrophes naturelles, inflation (Likert), stratégies d'adaptation |
| H | Besoins et perspectives : obstacles, formation, crédit, formalisation, classement de 3 priorités |
| I | Observations enquêteur : fiabilité des réponses, notes libres |

---

## Fonctionnalités techniques

### Code formulaire auto-généré

Les enquêteurs ne saisissent pas de code manuellement. Un champ `calculate` le génère à partir de la commune sélectionnée et de la date de collecte :

```
concat('OUEST-', ${commune}, '-', ${date_enquete})
```

Un champ `note` affiche le résultat à l'écran pour que l'enquêteur puisse le reporter dans son registre physique.

### Filtre en cascade sur les sous-secteurs (`choice_filter`)

La question C2 affiche uniquement les sous-secteurs correspondant au secteur sélectionné en C1. La feuille `choices` porte une colonne `secteur` qui tague chaque option à son parent. Choisir "Transport" filtre vers moto-taxi, tap-tap et camion. Choisir "Artisanat" filtre vers couture, menuiserie, ferronnerie et poterie. Vingt-quatre sous-secteurs en tout, cinq secteurs parents.

### Calculs et notes d'alerte

La marge brute mensuelle se calcule en arrière-plan et s'affiche en temps réel dans une note :

```
${revenu_mensuel_htg} - ${charges_mensuelles_htg}
```

Le ratio dette/revenu déclenche une note d'alerte conditionnelle lorsqu'il dépasse 3x, signalant une vulnérabilité financière critique à l'enquêteur sans interrompre la saisie.

### Groupe répété (Section F)

Les activités secondaires utilisent un `begin_repeat` dont le nombre d'itérations dépend d'un champ entier renseigné par le répondant, borné entre 1 et 3. Chaque itération capture le secteur, le revenu mensuel estimé et le temps hebdomadaire consacré.

### Logique de saut (`relevant`)

Trente-deux champs portent une condition `relevant`. Le nom de la banque n'apparaît que si le répondant possède un compte. Le détail des services mobile money n'apparaît que si l'usage est confirmé. Le montant de la dette et sa source n'apparaissent qu'après confirmation de l'endettement. Le groupe entier des activités secondaires reste masqué jusqu'à confirmation de leur existence.

### Contraintes et messages d'erreur bilingues

Les champs numériques portent des contraintes de plage avec messages d'erreur en français et en créole. L'âge est borné entre 18 et 80 ans. Les charges mensuelles doivent être inférieures au revenu déclaré, ce qui bloque les erreurs de saisie à la source plutôt qu'en phase de nettoyage. Les personnes à charge financière (B6) ne sont pas plafonnées au ménage, parce que les opérateurs informels haïtiens soutiennent fréquemment des enfants ou des proches vivant ailleurs.

---

## Choix de conception

**Le sol et le sabotaj** sont traités comme une catégorie d'instrument financier à part entière, distincte de la microfinance et du crédit bancaire. Dans l'économie informelle haïtienne, les tontines rotatoires constituent un mécanisme de liquidité primaire pour les petits opérateurs. Les ignorer fausse systématiquement les données d'accès au financement.

**Les personnes à charge hors ménage** (B6) sont comptabilisées explicitement. Un vendeur ambulant qui envoie de l'argent à ses enfants scolarisés en province, ou qui soutient un parent dans une autre commune, porte une charge financière réelle sur son activité. Les enquêtes ménage standard ne captent pas ce flux.

**L'insécurité et les catastrophes naturelles** sont documentées séparément en Section G. En Haïti, les deux mécanismes d'impact sont distincts : l'insécurité force la fermeture, le déplacement et la perte de clientèle ; une catastrophe détruit les stocks et l'infrastructure. Fusionner les deux dans une variable "choc" efface cette différence dans les données.

---

## Structure du fichier

```
enquete_eco_informelle_ouest_fad.xlsx
├── survey     (92 lignes, 15 colonnes)
├── choices    (155 lignes, 5 colonnes dont secteur pour choice_filter)
└── settings   (form_title, form_id, default_language, style, version)
```

Déployé sur ONA via Enketo. Compatible KoBoToolbox sans modification.

---
---

## Kontèks

Depatman Lwès kenbe pi gwo pati aktivite ekonomik enfòmèl Ayiti. Douz komin li yo, depi Pòtoprens rive Leyogàn pase pa Kwadèboukè, gen santèn milye operatè ki pa anrejistre ofisyèlman : machann raje, chofè moto-taksi, koutirye, moun k ap repare elektwonik, moun k ap vann manje. Travayè sa yo pa parèt nan okenn baz done enstitisyonèl e yo toujou rete deyò pèrimèt ankèt antrepriz estanda yo.

Pwojè sa a konstwi yon enstriman kolèk done sou teren pou dokimante kondisyon ekonomik reyèl yo : revni, chaj, aksè finansman (bank, lajan mobil, sòl ak sabotaj), ris chòk, ak bezwen devlopman. Sib la se 1 200 moun k ap reponn, 100 pou chak komin, ak yon plis nan zòn ki gen pi plis aktivite enfòmèl tankou Pòtoprens, Delma, Petyonvil, Kafou, ak Site Solèy.

Fòm nan nan 2 lang fransè-kreyòl ayisyen pou reponn kondisyon reyèl sou teren, kote ankètè yo ka chanje lang dapre pwofil moun k ap reponn nan.

---

## Achitekti fòm nan

Fichye XLSForm la gen 92 ranje nan fèy `survey`, 155 opsyon nan fey `choices` sou 21 lis diferan, ak yon fey `settings` ki konfigire pou ONA.

| Seksyon | Kontni |
|---|---|
| A | Idantifikasyon anketè, chwazi komin, GPS, kòd otomatik |
| B | Pwofil moun k ap reponn : sèks, laj, edikasyon, sitiyasyon matrimonyal, gwosè kay, moun ki depann de li|
| C | Aktivite prensipal : sektè, sou-sektè (filtré), kote, estati fòmèl, ane eksperyans, anplwaye |
| D | Revni ak chaj : estimasyon revni mwayen, chaj, maj brit (kalkile), revni ki bese |
| E | Aksè ak finansman : kont bank, lajan mobil, sòl/sabotaj, dèt ak rapò dèt/revni |
| F | Aktivite segondè : gwoup repete, jiska 3 fwa |
| G | Enpak chòk : ensekirite, katastwòf natirèl, enflasyon (Likert), estrateji adaptasyon |
| H | Bezwen ak pèspektiv : obstak, fòmasyon, kredi, fòmalizasyon, klase 3 bezwen prensipal |
| I | Obsèvasyon ankètè : fiabilite repons, nòt lib |

---

## Fonksyonalite teknik

### Kòd fòm otomatik

Anketè yo pa tape kòd manyèlman. Yon espas `calculate` jenere li otomatikman apati komin ki te chwazi a ak dat kolèkt la :

```
concat('OUEST-', ${commune}, '-', ${date_enquete})
```

Yon espas `note` montre rezilta a sou ekran pou ankètè a ka note l nan rejis fizik li.

### Filtre sou-sektè (`choice_filter`)

Kesyon C2 montre sèlman sou-sektè ki koresponn ak sektè ki te chwazi nan C1. Fèy `choices` la gen yon kolòn `secteur` ki make chak opsyon ak kategori li. Chwazi "Transpò" montre moto-taksi, tap-tap ak kamyon. Chwazi "Atizana" montre koutirye, ebenis, fèronri ak potri. Venn-kat sou-sektè an total, senk sektè paran.

### Kalkil ak nòt alèt

Maj brit chak mwa kalkile otomatik epi parèt an tan reyèl nan yon `note` :

```
${revenu_mensuel_htg} - ${charges_mensuelles_htg}
```

Rapò dèt/revni deklanche yon nòt alèt kondisyonèl lè li depase 3x, ki avèti ankètè a san entèwonp sezi done a.

### Gwoup repete (Seksyon F)

Aktivite segondè yo itilize yon `begin_repeat` ki depann de kantite aktivite moun k ap reponn nan ranpli, li ant 1 ak 3. Chak iterasyon kolekte sektè, revni mwayen estimé, ak tan chak semèn ou konsake pou aktivite a.

### Lojik sote (`relevant`)

Trannde espas gen yon kondisyon `relevant`. Non bank lan parèt sèlman si moun nan gen yon kont. Detay sèvis lajan mobil parèt sèlman si itilizasyon konfime. Montan dèt ak sous li parèt sèlman apre konfirmasyon endèteman. Gwoup aktivite segondè a rete kache jouk yo konfime yo egziste.

### Kontrènt ak mesaj erè bilenng

Espas nimerik yo gen kontrènt ak mesaj erè an fransè ak an kreyòl. Laj la borné ant 18 ak 80 an. Chaj chak mwa dwe mwens pase revni deklare a, sa ki bloke erè sezi nan sous olye pandan netwayaj. Moun ki depann finansyèman (B6) pa gen plafon ki mare ak gwosè kay la, paske operatè enfòmèl ayisyen souvan sipòte pitit oswa fanmi ki rete yon lòt kote.

---

## Chwa konsepsyon

**Sòl ak sabotaj** trete kòm yon kategori enstriman finansyè apa, diferan de mikwofinans ak kredi bank. Nan ekonomi enfòmèl Ayiti, tontinn rotatwa se yon mekanis likidite prensipal pou ti operatè. Pa mete yo fose done aksè finansman yo.

**Moun k ap depann deyò kay la** (B6) konte. Yon machann ki voye lajan ban pitit li ki ap etidye nan pwovens, oswa ki sipòte yon fanmi nan yon lòt komin, pote yon chaj finansyè reyèl sou aktivite li. Ankèt kay estanda yo pa kaptire fli sa a.

**Ensekirite ak katastwòf natirèl** dokimante separeman nan Seksyon G. An Ayiti, de mekanis enpak sa yo diferan : ensekirite fòse fèmti, deplase moun ak pèdi kliyan ; yon katastwòf detwi estòk ak enfrastrikti. Mete yo ansanm nan yon sèl varyab "chòk" efase diferans sa a nan done yo.

---

## Estrikti fichye a

```
enquete_eco_informelle_ouest_fad.xlsx
├── survey     (92 ranje, 15 kolòn)
├── choices    (155 ranje, 5 kolòn ki gen ladan secteur pou choice_filter)
└── settings   (form_title, form_id, default_language, style, version)
```

Deplwaye sou ONA via Enketo. Konpatib ak KoBoToolbox san modifikasyon.