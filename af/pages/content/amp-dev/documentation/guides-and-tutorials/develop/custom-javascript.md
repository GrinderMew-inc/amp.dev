---
"$title": Gebruik persoonlike JavaScript in AMP-bladsye
"$order": '7'
formats:
- webwerwe
author:
- moerse
contributors:
- CrystalOnScript
- fstanis
description: "'N Gids vir die gebruik van amp-script, 'n AMP-komponent waarmee u JavaScript kan skryf"
---

`amp-script` laat u toe om u eie JavaScript te skryf en uit te voer op 'n manier wat die prestasiewaarborge van AMP handhaaf. Die meeste AMP-komponente maak algemene interaksies deur middel van hul eie logika moontlik, sodat u u bladsy vinnig kan opbou sonder om JavaScript te skryf of biblioteke van derdepartye in te voer. Deur `amp-script` , kan u aangepaste logika aanvaar vir spesifieke gebruiksgevalle of unieke behoeftes sonder om die voordele van AMP te verloor.

Hierdie gids gee agtergrond oor hierdie komponent en beste praktyke vir die gebruik daarvan.

## Webwerkers

Oormatige JavaScript kan webwerwe stadig en reageer. Om te bepaal wat JavaScript-AMP-bladsye laai en wanneer dit uitgevoer word, verbied AMP se valideringsreëls ontwikkelaars om JavaScript op 'n webblad via 'n `<script>` -tag te gebruik.

[Webwerkers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) bied 'n manier om JavaScript veiliger te gebruik. Gewoonlik loop alle JavaScript [in 'n enkele draad](https://www.youtube.com/watch?v=cCOL7MC4Pl0) , maar elke werker loop in 'n draad van sy eie. Dit is moontlik omdat hulle nie toegang tot die DOM of die `window` , en elke werker in sy eie globale omvang loop. Hulle kan dus nie inmeng met mekaar se werk of met mutasies wat deur kode in die hoofdraad veroorsaak word nie. Hulle kan slegs met die hoofdraad en met mekaar kommunikeer via [boodskappe wat voorwerpe bevat](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope/postMessage) . Werkers bied 'n pad na 'n multidraad-web, 'n manier om JavaScript in 'n sandbox in te vou waar dit nie die UI kan blokkeer nie.

Werkers het nie toegang tot die DOM nie. Om hierdie leemte te vul, het die AMP-span 'n open-source biblioteek met die naam [WorkerDOM geskep](https://github.com/ampproject/worker-dom/) . WorkerDOM kopieer die DOM na 'n virtuele DOM en stel die kopie aan 'n werker beskikbaar. WorkerDOM herskep ook 'n deelversameling van die standaard DOM API. Wanneer die werker veranderinge aanbring in die virtuele DOM, herskep WorkerDOM die veranderinge in die regte DOM. Dit laat die werker die DOM manipuleer en veranderings aanbring op die bladsy met behulp van standaardtegnieke. Die DOM-sinkronisasie gaan net in een rigting. As die hoofdraad die DOM verander, bestaan daar geen meganisme om die werker daarvan in kennis te stel nie.

`amp-script` is 'n omslag rondom WorkerDOM wat WorkerDOM bruikbaar maak in AMP, verbindings bied met AMP-funksies, die oprigting van die ontwikkelaar-API en die instelling van beperkings wat die gebruikerservaring beskerm. WorkerDOM bied die kern van `amp-script` se funksionaliteit.

## Oorsig van `amp-script`

Die JavaScript-taal is dieselfde by 'n werker as elders in die blaaier. U kan dus in `amp-script` al die gewone konstrukte gebruik wat JavaScript bied. WorkerDOM herskep ook baie algemene DOM API's en stel dit beskikbaar vir u gebruik. Dit ondersteun algemene web-API's soos `Fetch` en `Canvas` , en dit gee u geselekteerde globale voorwerpe soos `navigator` en `localStorage` . U kan hanteerders op die gewone manier toewys vir blaaiergebeurtenisse.

`amp-script` ondersteun egter nie die hele DOM API of Web API nie, aangesien dit sy eie JavaScript te groot en omslagtig sal maak. Raadpleeg [die dokumentasie](../../../documentation/components/reference/amp-script.md#supported-apis) vir meer inligting, en verwys na [hierdie voorbeelde](https://amp.dev/documentation/examples/components/amp-script/) om die `amp-script` in gebruik te sien.

`amp-script` vervang 'n handvol sinchrone DOM API-metodes met alternatiewe wat 'n belofte lewer. In plaas van `getBoundingClientRect()` , gebruik u byvoorbeeld `getBoundingClientRectAsync()` . Soms is dit nodig vir DOM-API's wat sinchroniese toegang tot berekende uitleg bied, en soms sodat die `amp-script` die oorspronklike blaaiermetode kan skakel en op 'n antwoord kan wag.

Om die waarborge van AMP vir prestasie en uitlegstabiliteit te handhaaf, hou `amp-script` sekere beperkings in. As die grootte van die `amp-script` houer nie vasgestel is nie, kan u kode slegs 'n mutasie maak as dit deur 'n gebruikersinteraksie veroorsaak word. U kan nie stylblaaie of bykomende skrifte by die DOM voeg nie, en [`importScripts()`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope/importScripts) word nie ondersteun nie. Sien [die dokumentasie](../../../documentation/components/reference/amp-script.md#user-gestures) vir meer inligting.

## Die gebruik van JavaScript-raamwerke

Aangesien die DOM API nie ten volle ondersteun word nie, hoe is dit die beste om die DOM in `amp-script` manipuleer? Hier is twee moontlikhede.

**1) Weet wat ondersteun word.** Leer [die ryk stel ondersteunde API's ken](https://github.com/ampproject/worker-dom/blob/main/web_compat_table.md) . U moet anders dink aan die DOM API - minder as 'n groot stel eienskappe en metodes wat oor baie jare ontwikkel is, en meer as 'n bondige stel gereedskap.

**2) Gebruik Preact.** [React](https://reactjs.org/) is nie net 'n gewilde manier om webwerwe te bou nie, maar dit verander die DOM met behulp van 'n subset van die DOM API. `amp-script` kan volledige ondersteuning vir hierdie gedeelte van die API insluit, en dus volle ondersteuning vir React. Dit gesê: React-bundels oorskry die [limiet van 150](../../../documentation/components/reference/amp-script.md#size-of-javascript-code) `amp-script` . Dit word dus aanbeveel dat u [Preact](https://preactjs.com/) gebruik, 'n liggewig alternatief vir React. Preact is ontwerp vir eenvoudige migrasie vanaf React. Met Preact moet u uitgebreide interaksies kan opbou met minder bekommernis oor wat ondersteun word.

Die span het `amp-script` getoets met raamwerke soos [Vue](https://vuejs.org/) , [Angular](https://angularjs.org/) , [Aurelia](https://aurelia.io/) en [lit-html](https://lit-html.polymer-project.org/) , maar minder uitgebreid. As u 'n leemte vind, dien u ['n probleem in](https://github.com/ampproject/worker-dom/issues) - of, beter nog, [dien 'n trekversoek in](https://github.com/ampproject/worker-dom/pulls) .

Aangesien `amp-script` nie die DOM API volledig kan ondersteun nie, sal die kopiëring van 'n biblioteek soos [jQuery](https://jquery.com/) na 'n komponent `<amp-script>` nie werk nie.

## Gebruik gevalle

`amp-script` brei die funksie van u AMP-webblad uit. Aangesien dit 'n deelversameling van die DOM- en Web-API's ondersteun, en omdat dit [beperkend is met](https://amp.dev/documentation/components/amp-script/#restrictions) die gebruik daarvan, is dit nie 'n alledaagse JavaScript-oplossing nie. Enige groot hoeveelheid bestaande JavaScript is waarskynlik nodig om gewysig te word om in die `amp-script` konteks te werk.

`amp-script` bied egter 'n uitstekende manier om te sorg vir logika en interaksies wat bestaande AMP-komponente nie bied nie. Die volgende is 'n paar uitstekende gebruiksgevalle.

### Skep nuwe interaksies

`amp-script` gee u die krag van JavaScript en die DOM API. Hiermee kan u interaksies skep wat ander AMP-komponente nie kan nie, wat die deur oopmaak vir die volledige kreatiwiteit van die internet.

Dit gesê, voordat u na `amp-script` om 'n nuwe interaksie te skep, moet u kyk of 'n AMP-komponent of 'n kombinasie van komponente dieselfde kan doen. Deur gebruik te maak van bestaande AMP-komponente, word u kode uiteindelik kleiner en meer instandhoudend. As 'n komponent 'n effek kan bereik wat soortgelyk is aan wat u wil hê, kan u <a href="#enhance-amp-components" data-md-type="link">die gedrag moontlik met die `amp-script` aanpas</a> .

If you use `amp-script` to create an interaction that might be of interest to other developers, please consider [suggesting or contributing](https://github.com/ampproject/amphtml/blob/master/CONTRIBUTING.md) a new component.

### Voeg gevorderde logika by

As u logika nie netjies in 'n kompakte uitdrukking pas nie, is `amp-script` die beste keuse. `amp-bind` laat u toe om logika in gebruikersinteraksies in te voer, maar u JavaScript moet in een enkele uitdrukking pas. Omdat die kode in 'n HTML-attribuut vervat is, sal dit nie die voordeel hê van sintaksmarkering in u IDE nie; u kan nie breekpunte instel nie, en foutopsporing kan 'n uitdaging wees. In sulke gevalle kan `amp-script` u die beste programmeringspraktyke volg.

### Verbeter AMP-komponente<a name="enhance-amp-components"></a>

`amp-script` het toegang tot AMP-staat veranderlikes en die `AMP.getState()` en `AMP.setState()` metodes. Dit bied 'n pad om bestaande AMP-komponente met u eie logika te verbeter. Dit maak dit ook moontlik om die DOM buite die komponent `<amp-script>` te beïnvloed. Kyk [hier](https://amp.dev/documentation/examples/components/amp-script/#interacting-with-%3Camp-state%3E) en [hier](https://amp.dev/documentation/examples/components/amp-script/#interacting-with-amp-components) vir voorbeelde.

### Vervang amp-bind en amp-list

As u nog nie van AMP gebruik is nie en u gemaklik is met JavaScript, kan dit aanloklik wees om `amp-script` vir elke blaaierinteraksie te gebruik. Vir eenvoudiger interaksies sal u waarskynlik wil leer om `amp-bind` en die AMP se [aksies- en gebeurtenisstelsel te](../learn/amp-actions-and-events.md) gebruik. Vir meer uitgebreide interaksies, of vir gevalle waar meer logika en ingewikkelde manipulasie van toestande veranderlik is, sal `amp-script` [waarskynlik makliker wees](#when-to-replace-amp-bind-and-amp-list) .

### Hanteer bedienerdata

Met AMP kan u bedienergegewens ophaal met behulp van `amp-list` en dit formatteer met [`amp-mustache`](../../examples/documentation/amp-mustache.html) . In gevalle waar snagsjablone onvoldoende is, kan `amp-script` die data haal, formateer en die geformateerde data in die DOM spuit. As u bedienerdata moet masseer voordat u dit na `amp-mustache` , kan 'n `amp-script` funksie die databron wees vir die `amp-list` . Raadpleeg [die dokumentasie](../../../documentation/components/reference/amp-list.md?format=websites#initialization-from-amp-state) vir besonderhede en 'n kode-voorbeeld.

### Stel nuwe vermoëns bekend

You can use `amp-script` to leverage areas of the Web API and DOM API that aren't currently accessible to AMP components, or to use these APIs in ways that AMP components don't support. For example, `amp-script` supports `WebSockets` ([example](https://amp.dev/documentation/examples/components/amp-script/#using-a-websocket-for-live-updates)), `localStorage`, and `Canvas`. It supports a wide variety of browser events, so you can listen for events beyond [those that AMP passes to traditional components](https://amp.dev/documentation/guides-and-tutorials/learn/amp-actions-and-events/). And since `amp-script` provides access to the `navigator` object, you can retrieve information about [the user's browser](https://amp.dev/documentation/examples/components/amp-script/#detecting-the-operating-system) or [preferred language](https://amp.dev/documentation/examples/components/amp-script/#personalization).

## Wanneer moet u amp-bind en amp-list vervang?<a name="when-to-replace-amp-bind-and-amp-list"></a>

Vir 'n nuwe AMP-ontwikkelaar wat gemaklik is met JavaScript, lyk dit miskien makliker om altyd `amp-script` as om `amp-bind` en `amp-list` . In sommige gevalle pas `amp-bind` en `amp-list` egter beter.

`amp-bind` is oor die algemeen eenvoudiger vir basiese interaksies, waar die strakke integrasie in HTML-etikette dwingend is. In hierdie voorbeeld verander die teksfragment as u op 'n knoppie druk. AMP se databinding maak dit reguit en maklik om te lees:

```html
<p [text]="name">Rajesh</p>
<button on="tap:AMP.setState({name: 'Priya'})">I am Priya</button>
```

Net so, as u 'n API gebruik waarvan u die uitvoer beheer, kan u besigheidslogika op die bediener implementeer. U kan dalk die data wat die API uitvoer, formateer sodat dit glad in 'n `amp-mustache` sjabloon pas. `amp-list` is 'n goeie pas in sulke gevalle.

`amp-bind` bied ook 'n eenvoudige meganisme om tussen AMP-komponente te kommunikeer. In die voorbeeld stel die tik op 'n afbeelding in 'n `<amp-selector>` die statusveranderlike `selectedSlide` op `0` , wat weer 'n `<amp-carousel>` beweeg na sy eerste skyfie.

```html
<amp-carousel slide="selectedSlide">
...
</amp-carousel>

<amp-selector>
  <amp-img on="tap:AMP.setState({selectedSlide: 0})"/>
</amp-selector>
```

Tradisionele interaktiewe AMP-komponente kan ook meer geskik wees vir interaksies wat oor groot dele van 'n webblad strek - omdat u dalk nie soveel van die DOM in 'n `<amp-script>` wil toedraai nie. `amp-list` en `amp-bind` is styf geïntegreer met die res van AMP, wat dit gemaklik maak om binding oral op 'n webblad te gebruik.

Op bladsye wat meer ingewikkelde toestandsveranderlikes of meerdere interaksies behels, sal die gebruik van `amp-script` egter lei tot 'n eenvoudiger en meer instandhoudende kode. Neem hierdie voorbeeld van die [AMP Camp e-commerce demo-werf](https://camp.samples.amp.dev/product-details?categoryId=women-shirts&productId=79121) :

```html
<amp-selector
  name="color"
  layout="container"
  [selected]="product.selectedColor"
  on="select:AMP.setState({
      product:
        {
          selectedSlide: product[event.targetOption].option - 1,
          selectedColor: event.targetOption,
          selectedSize: product[event.targetOption].sizes[product.selectedSize] != null ?
                        product.selectedSize :
                        product[event.targetOption].defaultSize,
          selectedQuantity: 1
        }
      })"
></amp-selector>
```

Hierdie demo is geskep voordat die `amp-script` vrygestel is. Maar hierdie soort logika sou makliker wees om in JavaScript te skryf en te ontfout. Vir bladsye met meer besigheidslogika, kan die gebruik van `amp-script` u in staat stel om verwarring te voorkom en beter programmeringspraktyke te volg.

In baie gevalle wil u beide `amp-script` en `amp-bind` op dieselfde bladsy gebruik. Ontplooi `amp-bind` vir eenvoudiger interaksies, draai na `amp-script` as u meer logika of struktuur benodig. Verder, hoewel `amp-script` slegs mutasies kan maak aan sy DOM-kinders, [soos hierbo aangedui](#enhance-amp-components) , kan dit die res van die bladsy beïnvloed deur toestandsveranderlikes te verander. `amp-bind` doen die res, soos in [hierdie voorbeeld](https://amp.dev/documentation/examples/components/amp-script/#interacting-with-%3Camp-state%3E) .

Alhoewel WorkerDOM die regte DOM sal verander wanneer u kode die virtuele DOM waartoe dit toegang het, verander, bestaan daar geen meganisme om in die omgekeerde rigting te sinkroniseer nie. Dit is dus nie raadsaam om `amp-bind` of ander maniere te gebruik om die kinders van u `<amp-script>` . Reserveer die gedeelte van die bladsy vir u JavaScript `amp-script` .

## Dra by tot `amp-script`

`amp-script` ontwikkel altyd, net soos AMP. U kan help om dit te verbeter deur betrokke te raak. Dink aan nuwe funksies wat ander ontwikkelaars ook nodig het, [stuur probleme](https://github.com/ampproject/amphtml/issues) uit en gee natuurlik [nuwe funksies voor en dra dit by](https://github.com/ampproject/amphtml/pulls) !
