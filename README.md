# personal-vue-prj

### Introduction

This application is made using vue.js.

It is used to get an insight of lego sets, you can get an overview, and also dive into details using the tables and hover events on the web page.
The standard settings on this application gives you insight in the amount of pieces in a lego set, the year it has been released.
The line chart gives you information about the average amount of pieces in one set per year. 

### Using the app

In order to use the app on your local computer, you will have to clone this repository.

To do that, in terminal run this command:

``` git clone https://github.com/mandemt/personal-vue-prj.git```


open the file and go to the folder `/mandemt-vue`.

In this folder, run `npm run dev`. The application should start on your localhost that is also presented in the terminal.

Once you opened the web page. It should show a screen without any information. This is because there is an API key missing.
This is a key you can request by creating an account en verifying it on the site of the API creator.

You can to that here: <a href="https://rebrickable.com/api/v3/docs/?key=0ec2f10aec8ef5b1fb6c968828b665b6"> Rebrickable API manual</a>

This key is a string of numbers and letters. This should be in a hidden file using the format .env.



You can call the file just `.env` and put the following code in: 
``` 
VITE_API_KEY=[your key]
```
The brackets are not in the actual file.

When selecting a theme, a bar graph and line graph should appear. You can filter this data in the second dropdown menu, and sort in the third dropdown menu. Notice that when you <b>filter</b> on a specific year, you can no longer <b>sort</b> on year.

To make changes to the api fetch, like file size or you want other values. You can make changes to the file `Rebrickable.vue` If you want to change the
bar or line chart, you can make most changes to `Computed.vue`

The fetching of data happens in the first file. Here it gets filtered and averages are calculated. If you want different values, for example urls or parent theme ids, you can change these, but make sure you also change these values in the `Computed.vue` file so that the bars still generate.

For more information, you can use my contact information.


##### Tom Mandemaker, 2022  | Rebrickable API  https://rebrickable.com/api/v3/docs
