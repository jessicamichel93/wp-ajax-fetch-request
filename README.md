# wp-ajax-fetch-request
A step by step guide how to build a wp ajax request with fetch() - made for own use, written in dutch
Stap voor stap uitleg om een wordpress ajax call te bouwen met fetch() ES6.

## Benodigheden
* Endpoint aanmaken
* ajax php file aanmaken die include wordt door de functions.php
* JS / TS file voor de fetch() request
* Archive page (opgebouwde pagina met posts die met ajax later ingeladen moeten worden)

## Basis opbouwen
We maken een los bestand aan die geinclude wordt door de functions.php
Hierin gaan we met de standaard wordpress functie aan de slag om een basis request te maken. Die geven we een callback mee, wat er zo uitziet

```HTML
add_action('rest_api_init', function () {
    register_rest_route('mywebsite/v1', '/projectfilters', [
        'methods' => 'GET',
        'callback' => 'get_projectfilters_results',
    ]);
});

function get_projectfilters_results($request) // $data krijg je terug van je call => request
{
return ['data' => 'data',];
}
```
