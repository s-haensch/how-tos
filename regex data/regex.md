# Regular Expressions for Data Journalists

- Regex Pattern need to have an entry point (e.g. line start, certain character, ...)

## DATA EXTRACTION
### twitter 
mario götze tweets


```js
649260248881045504,"#Oktoberfest @FCBayern  Seid ihr auch schon auf der Wiesn gewesen? Schönen Abend noch! #PartOfMario pic.twitter.com/bwg2gsCJt6"
648968301088010240,"#FCBDIN 5:0 Yeeeees!!! @FCBayern @ChampionsLeague  6 Points/8:0 Goals - looks great! What do you think?!"
646724805107052544,"Great afternoon @Audi @David_Alaba - Who are the guys with the helmet??  look great!  @FCBayern  #PartOfMario pic.twitter.com/CQGKL62LFj"
```

__get all user tags__  
`\@[a-Z\d_]+`

__get all lines without user tags__  
`^(?!.*\@[a-Z\d_]+).*`

__get everything after " and before user tag__  
`(?<=").*?(?=\@)`

__get user tag not followed by user tag (last in line) and rest of line__  
`(\@[a-Z\d_]+)(?!.*\@[a-Z\d_]+).*(?=.*$)`

## DATA CLEANING
```js
Geography                Urban Area                  Population
Japan                    Tokyo-Yokohama              37,750,000
Indonesia                Jakarta                     31,320,000
India                    Delhi                       25,735,000
South Korea              Seoul-Incheon               23,575,000
Philippines              Manila                      22,930,000
India                    Mumbai                      22,885,000
Pakistan                 Karachi                     22,825,000
China                    Shanghai                    22,685,000
United States            New York                    20,685,000
Brazil                   Sao Paulo                   20,605,000
China                    Beijing                     20,390,000
Mexico                   Mexico City                 20,230,000
China                    Guangzhou-Foshan            18,760,000
Japan                    Osaka-Kobe-Kyoto            16,985,000
Russia                   Moscow                      16,570,000
```


### world population data
tasks:
- turn into csv
- change number format 20,685,000  to 20,685.00

steps
1. __get all whitespace of more than two whitespace characters__  
`\s{2,}`  
replace with tabs `\t`
1. __get everything that's not a tab or line end__  
`([^\t\n]*)`  
enclose in quotations marks _(group together comma-notated numbers)_
1.  __select all tabs__  
`\t`  
replace with commas `,`
1. save as .csv
1. __get all ocurrances of three zeros after a comma__  
`,0{3}`
replace with `.00`


## Pitfalls 🙄
### Lazyness + Greediness

One thing to watch out for, is that by default the star and plus operator are greedy.
Having a list of street names like this
```js
const streetNames = [
  "Május1. tér",
  "Március 15. utca",
  "November 17. utca",
  // ...
];

// the result I got:
{
  "date": "5 Március",  // should be 15!
  "monthName": "march",
  "total": 83,
  "individual": [
    ["Március 15. utca", 50],
    ["Március 15. tér", 23],
    ["Március 15. sétány", 2],
    ["Március 15 utca", 1],
    ["Március 15-e tér", 1],
    ["Március 15. út", 5],
    ["Március 15-e utca", 1]
  ]
}
```
and my regex was this, searching for Hungarian month names in combination with one or two digits, meaning a date in the name
```js
[A-zÄäÖöÜüß ]*(január|február|március|április|május|június|július|augusztus|szeptember|október|november|december).*(\d{1,2}).*
```
in detail:
```js
/[A-zÄäÖöÜüß ]*/     // a possible bunch of non-digits at the start
(január|február|..)  // isolate a hungarian month name
.*                   // possibly something in between (dots, spaces, etc.)
(\d{1,2})            // isolate one or two digits
.*                   // possibly something at the end
```
Using it for the string `Március 15. utca`, it only gave me `5` as the isolated group item instead of `15` that I was expecting. The reason was the "possibly something in between" `.*` that came before the "one or two digit"-group `(\d{1,2})`. It nabbed away the first digit, because it was greedy and the condition was met anyway. The solution was to set it to lazy by using `.*?`. With the question mark at the end, it would not take away the number from my number group.



## Regex and .js

#### dynamic regular expressions
```js
const monthsGrench: [
  "Janvier",
  "F[ée]vrier",
  "Mars",
  "Avril",
  "Mai",
  "Juin",
  "Juillet",
  "Août",
  "Septembre",
  "Octobre",
  "Novembre",
  "Décembre"
];
const monthsGerman: [
  "Januar",
  "Februar",
  "März",
  "April",
  "Mai",
  "Juni",
  "Juli",
  "August",
  "September",
  "Oktober",
  "November",
  "Dezember"
];

const letters = "A-zÄäÖöÜüß ";

// create regular expressions dynamically
let regular = new RegExp(
  `[${letters}]+(\\d{1,2}).*(${monthsGerman.join("|")}).*`,
  "i"
);

regular = new RegExp(
  `[${letters}]+(\\d{1,2}).*(${monthsFrench.join("|")}).*`,
  "i"
);
```