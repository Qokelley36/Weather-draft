# Weather-draft
#Weather draft


import requests
import json
import time

print('Welcome to the Weather forecast! Here you can get the current weather data for most major cities.\n'
      "\nFor a weather forecast, please enter the 5 digit U.S. Zip Code for your desired location \n"
      "or the city's name, comma, and 2-letter country code, examples are below.\n"
      'inputs made without a country code will return inaccurate results.\n'
      "Examples are... Oxford, GB - Las Vegas, US - Moscow, RU... \n")


def main():
    #Main function for the program, allows user to input a zip code or city to receive forecast
    url = 'https://api.openweathermap.org/data/2.5/weather'
    url_ext = 'https://api.openweathermap.org/data/2.5/forecast'
    location = input('Please enter the Zip Code or City, Country code: ')
    while True:
        try:
            weather_current(location, url)
            weather_extended(location, url_ext)
            print('')
            more_weather()
            break
        except LookupError:
            print('')
            more_weather()
            break


def weather_current(location, url):
    #Makes a GET request to the url for current weather, verifies connection is made, displays the data
    if location.isdigit() is True:
        query_params = {'zip': location, 'APPID': '6665c0a36b87a550f86be63eccd6f70a'}
    else:
        query_params = {'q': location, 'APPID': '6665c0a36b87a550f86be63eccd6f70a'}
    response = requests.get(url, params=query_params, timeout=(5, 14))
    try_web(response, location)
    if response.status_code == 200:
        print('Successful... Location Found:')
    current_parsed = json.loads(response.text)
    current_formatted(current_parsed)


def weather_extended(location, url_ext):
    #Makes a GET request to the url for extended forecast, displays the data
    if location.isdigit() is True:
        query_params = {'zip': location, 'cnt': 16, 'APPID': '6665c0a36b87a550f86be63eccd6f70a'}
    else:
        query_params = {'q': location, 'cnt': 16, 'APPID': '6665c0a36b87a550f86be63eccd6f70a'}
    response = requests.get(url_ext, params=query_params, timeout=(5, 14))
    try_web(response, location)
    ext_parsed = json.loads(response.text)
    ext_formatted(ext_parsed)


def convert_temp(temp):
    #Converts to murica
    f_degree = round((((temp - 273.15)*9)/5)+32)
    c_degree = round(temp - 273.15)
    return f'{f_degree}{chr(176)}F / {c_degree}{chr(176)}C'


def try_web(response, location):
    #Try Except block to test the request was successful, additionally checking if the city or zip code entered is valid 
    try:
       response.raise_for_status()
    except requests.HTTPError as error0:
        if response.status_code == 404:
            if location.isdigit() is True:
                print(f"The zip code entered '{location}' could not be located or is not valid.")
            else:
                if location.__contains__(','):
                    print(f"The city entered '{location[0:-2].title() + location[-2:].upper()}' could not be located.")
                else:
                    print(f"The city entered '{location.title()}' could not be located.")
        else:
            print('we do not have access to single digit zip codes.')
            print(f'{error0}')
    except requests.ConnectionError as error1:
        print('Error Connecting')
        print(error1)
    except requests.Timeout as error2:
        print('Timeout Error')
        print(error2)
    except requests.RequestException as error3:
        print('Something Else Went Wrong')
        print(error3)


def current_formatted(parsed):
   
    #output of the current weather
    city = str(json.dumps(parsed['name'])).replace('"', '')
    country = str(json.dumps(parsed['sys']['country'])).replace('"', '')
    timezone = int(json.dumps(parsed['timezone']))
    epoch_time = int(json.dumps(parsed['dt']))
    true_time = epoch_time + timezone
    current_time = time.strftime("%A, %b %d, %Y %I:%M %p (local time)", time.gmtime(true_time))
    temp = float(json.dumps(parsed['main']['temp']))
    conditions = str(json.dumps(parsed['weather'][0]['description'])).replace('"', '').title()
    print(f'Weather forecast for {city}, {country} on {current_time}:\n'
          f'Current Temperature {convert_temp(temp)}\n'
          f'Current Conditions: {conditions}\n')


def ext_formatted(parsed):
    
    #output of the extended forecast
    print(f"{'36 Hour Forecast':30}{'Temperature':22}{'Conditions'}")
    #For loop to pull the data 
    for i in range(1, 15, 2):
        epoch_time = int(json.dumps(parsed['list'][i]['dt']))
        timezone = int(json.dumps(parsed['city']['timezone']))
        true_time = epoch_time + timezone
        future_time = time.strftime("%a, %b %d %I:%M %p", time.gmtime(true_time))
        temp = float(json.dumps(parsed['list'][i]['main']['temp']))
        conditions = str(json.dumps(parsed['list'][i]['weather'][0]['description'])).replace('"', '').title()
        print(f'{future_time:30}{convert_temp(temp):22}{conditions}')


def more_weather():
  
    option = str(input('Would you like to enter another weather forecast, Yes or No? ')).lower().strip()
    #while loop for a yes selection or to exit the program (and to catch input errors)
    while not (option == 'yes' or option == 'no'):
        option = str(input('You did not enter a valid selection.\n'
                           'Please enter Yes for another forecast or No to exit: ')).lower().strip()
    if option == 'yes':
        print('')
        main()
    if option == 'no':
        print('Thank you for using the weather forecast. Goodbye')


main()
