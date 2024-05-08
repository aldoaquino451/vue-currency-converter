Currency Converter
===

- container:

  - titolo

  - componente currency: 
    - valore iniziale (cercato) testo piccolo
    - valore finale testo grande

  - componente search:
    - input number (4 numeri dopo la virgola)
    - input select

  - componente chart
    - componente di chart.js (o api simili)


API Frankfurter: riceve due dati (input e select iniziali) e restituisce due dati (input e select finali)
https://www.frankfurter.app/docs/

lista currencies https://api.frankfurter.app/currencies
chiamata axios https://api.frankfurter.app/latest?amount=2&from=EUR&to=USD
data chart https://api.frankfurter.app/2023-01-01..?from=AUD&to=USD

#### 1
la chiamata axios dentro app
passo i dati dentro le componenti search e currency

