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
* We gaan even een hardcoded filter toevoegen en kijken of deze iets terug geeft in de console log

```JavaScript
  fetch("/wp-json/mywebsite/v1/photofilters?filter=activiteit", {
```

```
GET https://mywebsite.nl/wp-json/mywebsite/v1/photofilters?filter=activiteit 404 (Not Found)
{code: "rest_no_route", message: "Geen route gevonden die overeenkomt met de URL en aanvraagmethode.", data: {…}}
```

We krijgen 2 dingen terug, de console.log van de DATA van de fetch request en een 404 met de aangemaakte filter params. Het klopt dat we een 404 krijgen want we doen er nog niets mee.

* Nu gaan we parameters opbouwen en deze meesturen in de call

```HTML
function get_projectfilters_results($request) // data krijg je terug van je call => request
{
    $params = $request->get_params();

    return ['data' => $params,];
}
```
