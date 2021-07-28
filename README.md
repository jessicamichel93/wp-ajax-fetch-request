# wp-ajax-fetch-request
A step by step guide how to build a wp ajax request with fetch() - made for own use, written in dutch

Stap voor stap uitleg om een wordpress ajax call te bouwen met fetch() ES6, gemaakt voor eigen gebruik als naslagwerk.

## Benodigheden
* Endpoint aanmaken
* ajax php file aanmaken die include wordt door de functions.php
* JS / TS file voor de fetch() request
* Archive page (opgebouwde pagina met posts die met ajax later ingeladen moeten worden)

## Basis opbouwen
* We maken een los bestand aan die geinclude wordt door de functions.php
* Hierin gaan we met de standaard wordpress functie aan de slag om een basis request te maken. Die geven we een callback mee, wat er zo uitziet

```PHP
function get_projectfilters_results($request) // $request krijg je terug van je call => request
{
return ['data' => $request,];
}

add_action('rest_api_init', function () {
    register_rest_route('mywebsite/v1', '/photofilters', [
        'methods' => 'GET',
        'callback' => 'get_photofilters_results',
    ]);
});
```

* We gaan nu ook in onze aangemaakte JS of TS file, een globale fetch() request opbouwen. Let erop dat de fetch url overeenkomt met de naam van de aangemaakte register_rest_route. In mijn voorbeeld komen **mywebsite** en **photofilters** overeen. Bovenstaande werkt nog niet, maar het is een basis waar we uit verder werken. 

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

## POSTS doorsturen naar je $request
* We gaan nu onze callback functie verder uitbreiden, zodat we daadwerkelijk ook onze posts (van een cpt o.i.d) kunnen doorsturen naar de request.
* We defineren variabelen voor onze CPT en het aantal per pagina, dat maakt het iets overzichtelijker
* We maken $args aan die we vullen met args die we straks meegeven aan onze WP_QUery waarmee we onze posts opbouwen
* Vervolgens maken we een variabele posts met een lege array, deze vullen we straks met de data vanuit onze request
* En we defineren onze WP_Query
* Vervolgens loopen we d.m.v een foreach door onze posts en geven we elk van de posts de titel, post id en permalink mee (kan van alles zijn, maar voor dit voorbeeld even deze 3)
* Als laatste stap sturen we onze posts array door naar de request

```PHP
function get_projectfilters_results($request) // $request krijg je terug van je call => request
{
$postType = 'portfolio_type'; // In mijn voorbeeld heet mijn CPT portfolio_type
$postPerPage = 12;

$args = [
	'post_type' => $postType,
        'post_status' => 'publish',
        'posts_per_page' => $postPerPage,
];

$posts = [];
$query = new WP_Query($args);

    foreach ($query->posts as $post) {
        $post = (array)[
            'title' => $post->post_title,
            'post_id' => $post->ID,
            'permalink' => get_permalink($post->ID),
        ];
        $posts[] = $post;
    }

    return ['data' => $posts];
}

add_action('rest_api_init', function () {
    register_rest_route('mywebsite/v1', '/photofilters', [
        'methods' => 'GET',
        'callback' => 'get_photofilters_results',
    ]);
});
```

* Als we bovenstaande code hebben, de cpt juist is ingevuld, dan zouden we nu in onze console de post data terug moeten krijgen
```
{data: Array(8)}
data: (8) [{…}, {…}, {…}, {…}, {…}, {…}, {…}, {…}]
__proto__: Object
```

## Custom Filters
* We gaan even een hardcoded filter toevoegen in de fetch(), dat doen we door ?filter=activiteit toe te voegen.  maar dit kunnen ook bv alle terms zijn o.i.d. Maar we houden het nu even zo simpel mogelijk.

```JavaScript
  fetch("/wp-json/mywebsite/v1/photofilters/?filter=activiteit", {
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

NOTE: DIT KLOPT NIET, moet ik nog aanpassen.

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
Meer stappen binnenkort...
