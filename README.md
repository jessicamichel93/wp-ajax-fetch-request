# wp-ajax-fetch-request
A step by step guide how to build a wp ajax request with fetch() - made for own use, written in dutch

Stap voor stap uitleg om een wordpress ajax call te bouwen met fetch() ES6.

## Benodigheden
* Endpoint aanmaken
* ajax php file aanmaken die include wordt door de functions.php
* JS / TS file voor de fetch() request
* Archive page (opgebouwde pagina met posts die met ajax later ingeladen moeten worden)

## Basis opbouwen
* We maken een los bestand aan die geinclude wordt door de functions.php
* Hierin gaan we met de standaard wordpress functie aan de slag om een basis request te maken. Die geven we een callback mee, wat er zo uitziet

```PHP
add_action('rest_api_init', function () {
    register_rest_route('mywebsite/v1', '/photofilters', [
        'methods' => 'GET',
        'callback' => 'get_photofilters_results',
    ]);
});

function get_projectfilters_results($request) // $data krijg je terug van je call => request
{
return ['data' => 'data',];
}
```

* We gaan nu ook in onze aangemaakte JS of TS file, een globale fetch() request opbouwen. Let erop dat de fetch url overeenkomt met de naam van de aangemaakte register_rest_route. In mijn voorbeeld komen **mywebsite** en **photofilters** overeen.

```JavaScript
 const filterInit = () => {
 
// fetch function
const getData = () => {
	fetch('/wp-json/mywebsite/v1/photofilters', {
		method: 'GET',
     headers: {
       'Content-Type': 'application/json',
   },
  })
 .then((response) => response.json())
 .then((data) => console.log(data))
     .catch((error) => {
       console.log(error);
       throw new Error(error);
     });
 };
 
getData();
};
```

* in de callback functie in de theme-ajax.php gaan we straks de query opbouwen, taxonomys inladen, de post title, permalink etc inladen en de data terugsturen naar de fetch() function in een aparte js file later
* We gaan even een hardcoded filter toevoegen in de fetch(), dat doen we door ?filter=activiteit toe te voegen.  maar dit kunnen ook bv alle terms zijn o.i.d. Maar we houden het nu even zo simpel mogelijk.

```JavaScript
  fetch("/wp-json/mywebsite/v1/photofilters?filter=activiteit", {
```

* Nu gaan we de gebouwde parameters opbouwen en deze meesturen in de call (in dit geval nu even de hardcoded filter ?filter=activiteit.

```PHP
function get_projectfilters_results($request) // data krijg je terug van je call => request
{
    $params = $request->get_params();

    return ['data' => $params,];
}
```

Nu gaan we kijken of we daadwerkelijk al iets terugkrijgen in de console

```
GET https://mywebsite.nl/wp-json/mywebsite/v1/photofilters?filter=activiteit 404 (Not Found)
{code: "rest_no_route", message: "Geen route gevonden die overeenkomt met de URL en aanvraagmethode.", data: {…}}
```

We krijgen 2 dingen terug, de console.log van de DATA van de fetch request en een 404 met de aangemaakte filter params. Het klopt dat we een 404 krijgen want we doen er nog niets mee. De basis werkt dus tot nu toe.

## Filters uitlezen en DATA meesturen on Click
Als het goed is, heb je al een archive page met filters in de form van checkboxes o.i.d. Voor de uitleg gaan we even verder met de filters die ik had aangemaakt. We gaan nu de filters uitlezen en on Click de data meesturen naar de call. 

* Buiten de fetch() voegen we onze variabelen toe, een click functie op de filter labels en bouwen we een object op waar alle taxonomys en terms ingeladen worden.

```JavaScript
// Variables
const filters = document.querySelectorAll(".filters__item-label");
const filterObj = [];

filters.forEach((filter) => {
  filter.addEventListener("click", () => {
    const taxonomy = filter.dataset.tax; // inladen van de data attrubuten met dataset
    const term = filter.dataset.term;

    addFilterToObject(taxonomy, term);
		getData();
		// getData(); halen we onderaan het bestand weg en roepen we nu aan op elke klik

  });
});


const addFilterToObject = (key, value) => {
  if (key in filterObj) {
    if (filterObj[key].includes(value)) {
      filterObj[key] = filterObj[key].filter((filter) => filter !== value);
		 if (!filterObj[key].length) {
        delete filterObj[key];
      }
    } else {
      filterObj[key] = [...filterObj[key], value];
    }
  } else {
    filterObj[key] = [value];
  }

  console.log(filterObj);
};
```

* Nu we een filter object hebben met de taxonomy en terms, kunnen we verder gaan en onze parameters dynamisch inladen in de fetch request.

```JavaScript
const createParams = () => {
  // Met object.entries kunnen we door een object loopen
  Object.entries(filterObj).forEach(([key, value]) => {
    console.log(`${key} ${value}`); // "a 5", "b 7", "c 9"
  });
};
```

De fetch() wijzigen we dan naar de variable met de custom params
```JavaScript
  fetch(`/wp-json/mywebsite/v1/photofilters/${createParams}`, {
```

* Nu gaan we de custom query string opbouwen die als parameters ingeladen worden
```JavaScript

```
