# OSCF SDI Interface
Version 1.0.0

## Forenotices

Expect bugs! This is a rushed out quick implementation. If you find issues with this interface, I will provide the URLs for interacting with the SDI API below in due course. 

OSCF is the Open Scraper Crawler Framework - a project to enhance the infrastructure IFPI provides for web scraping. Please contact brodie.gordon-beaven@ifpi.org or Oscar.Ferreira@ifpi.org for further details. 

SDI is a component of OSCF (Simple Database Interactions). Apologies for all the acronyms - we will make it all a bit simpler eventually! 

You will need a GCPN account to use this. If you do not have a GCPN account, please make contact with IFPI's london development team. 

## Overview

This interface is designed to interact with OSCF's SDI API, allowing for the insertion of crawler results into GCPN and handling authentication to the API. The project is built using Python and leverages the `requests` library for HTTP requests and `loguru` for logging.

Any issues at this stage, please contact me at brodie.gordon-beaven@ifpi.org - suggestions, bugs, critisicm, cries for help - its all welcome!

## Quick Start Guide

### Prerequisites

- Python 3.10 or higher (match case is in use)
- `pip` for package management

### Installation

1. Clone the repository:

    ```sh
    git clone https://github.com/yourusername/sdi-project.git
    cd sdi_interface-project
    ```

2. Install the required packages:

    ```
    pip install git+https://Brodie-IFPI:TOKEN_PROVIDED_BY_BRODIE@github.com/Brodie-IFPI/OSCF_SDI.git#egg=sdi_interface
    ```

### Usage

As of 06/02/2025, there is only one function that can be used and is simplistic.

1. Import sdi and add your GCPN credentials below + the crawler name

    ```python
    from sdi_interface import SDI

    sdi = SDI(username='brodie@ifpi.org', password='pass', crawler_name='demo')
    ```

2. Simply insert a result. There are 2 compulsory fileds, and 3 optional fields. 

    URL and Filename are compulsory fields.
   
    ```python
    sdi.insert_crawler_result(url="www.infringement.com", filename="infringement")
    ```
    You can also include a referrer link, a description and an identifier for the infringement uploader

   ```python
    sdi.insert_crawler_result(url="www.infringement.com",
                               filename="infringement",
                               referrer="www.google.com"
                               description="Something I found"
                               uploader="Oscar")
    ```

   All API interactions are performed on a separate thread. There is no need to await for the thread to finish.
   
## Detailed Code Commentary

### `sdi.py`

This file contains the main `SDI` class, which handles authentication and the insertion of crawler results.

- **`__new__` and `__init__` Methods**: Ensures that `SDI` is a singleton class, meaning only one instance can exist at a time.
- **`setup` Method**: Authenticates the user and sets up the necessary headers for API requests.
- **`insert_crawler_result` Method**: Inserts a crawler result into the SDI database. This method validates the body content, sends the request, and processes the response.

### `services/api_contact.py`

This file contains classes and methods for interacting with the SDI API.

- **`Transaction` Class**: Handles the creation, data validation and sending of API requests.
- **`Digester` Class**: Processes the API responses and handles different status codes.
- **`Authenticator` Class**: Manages user authentication and token generation.

### `utils/general.py`

This file contains utility functions and decorators.

- **`force_thread` Decorator**: Runs a function in a separate thread.
- **`Utils` Class**: Provides utility methods, such as generating a default crawler name if one is accidentally not supplied.

## Upcoming and on my todos

There is alot to be done here - this is a simple implementation to get the ball rolling. The more urgent requirements include:
- Improved logging for informing on whether the crawler result insert was successful.
- General testing across the API / Facade - both user and automated
- Batch sending to the API rather than one by one
- Further endpoints for more visibility on whats going on inside GCPN and access to set lists of things e.g rep priorities or common filename filters
- Pypi support for installation (and threby greater securtiy)
- Clean up all the TODO's I left lying around, i.e fix the easy problems and ticket the hard ones
- Implement a middleware database between the API and your crawlers to hold crawler results.
- Automated endpoint updating i.e if we change a field on the API, you will not need to update this library as the endpoints on here will change too.
- Better response digesting - i.e take certain precautions if the API is throwing 500s.
