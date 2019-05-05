# capstone-project-week-3
{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Week 3 Clustering Neighborhoods Project pt.1"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Hello. This particular project is all about clustering neighborhoods in Toronto using the website of https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M where webscraping is utilized.\n",
    "\n",
    "However, the first step is to import the appropriate libraries for this project first."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {},
   "outputs": [],
   "source": [
    "import numpy as np \n",
    "import pandas as pd \n",
    "from bs4 import BeautifulSoup \n",
    "import requests "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Next, we prepare our web scraping code by utilizing BeautifulSoup."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "metadata": {},
   "outputs": [],
   "source": [
    "url='https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M'\n",
    "source = requests.get(url).content\n",
    "content = BeautifulSoup(requests.get(url).content, 'lxml')"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Project Rules\n",
    "\n",
    "The dataframe will consist of three columns: PostalCode, Borough, and Neighborhood.\n",
    "\n",
    "Only process the cells that have an assigned borough. \n",
    "\n",
    "Ignore cells with a borough that is Not assigned.\n",
    "\n",
    "More than one neighborhood can exist in one postal code area. \n",
    "\n",
    "For example, in the table on the Wikipedia page, you will notice that M5A is listed twice and has two neighborhoods: Harbourfront and Regent Park. These two rows will be combined into one row with the neighborhoods separated with a comma as shown in row 11 in the above table.\n",
    "\n",
    "If a cell has a borough but a Not assigned neighborhood, then the neighborhood will be the same as the borough. So for the 9th cell in the table on the Wikipedia page, the value of the Borough and the Neighborhood columns will be Queen's Park.\n",
    "\n",
    "Clean your Notebook and add Markdown cells to explain your work and any assumptions you are making.\n",
    "\n",
    "In the last cell of your notebook, use the .shape method to print the number of rows of your dataframe."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Here we actually webscrape the data from the wikipedia page\n",
    "table = content.find('table')\n",
    "td = table.find_all('td')\n",
    "postcode = []\n",
    "borough = []\n",
    "neighbourhood = []\n",
    "\n",
    "# create a list with the scraped data\n",
    "for i in range(0, len(td), 3):\n",
    "    postcode.append(td[i].text.strip())\n",
    "    borough.append(td[i+1].text.strip())\n",
    "    neighbourhood.append(td[i+2].text.strip())"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {},
   "outputs": [],
   "source": [
    "# create the actual DataFrame with the lists previously scraped and give the columns appropriate names  \n",
    "df_codes = pd.DataFrame(data=[postcode, borough, neighbourhood]).transpose()\n",
    "df_codes.columns = ['Postal Code', 'Borough', 'Neighborhood']"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Cleaning pt.1\n",
    "\n",
    "The next step requires us to follow some of the project rules requirements. In this case, it would be ignoring boroughs with the 'Not assigned' value. Also, if a cell has a borough but a 'Not assigned' neighborhood value, then the neighborhood value will be the same as the borough value for that particular row."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Ignore cells with a borough that is Not assigned.\n",
    "df_codes['Borough'].replace('Not assigned', np.nan, inplace=True)\n",
    "df_codes.dropna(subset=['Borough'], inplace=True)\n",
    "\n",
    "# Also, if a cell has a borough but a 'Not assigned' neighborhood value, \n",
    "# then the neighborhood value will be the same as the borough value for that particular row.\n",
    "df_codes['Neighborhood'].replace('Not assigned', \"Queen's Park\", inplace=True)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Cleaning pt. 2\n",
    "\n",
    "Next we do the last constraint which is combining neighborhoods if they exist in one postal code."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Postal Code</th>\n",
       "      <th>Borough</th>\n",
       "      <th>Neighborhood</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>M1B</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Rouge, Malvern</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>M1C</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Highland Creek, Rouge Hill, Port Union</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>M1E</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Guildwood, Morningside, West Hill</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>M1G</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Woburn</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>M1H</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Cedarbrae</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>5</th>\n",
       "      <td>M1J</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Scarborough Village</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>6</th>\n",
       "      <td>M1K</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>East Birchmount Park, Ionview, Kennedy Park</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>7</th>\n",
       "      <td>M1L</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Clairlea, Golden Mile, Oakridge</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>8</th>\n",
       "      <td>M1M</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Cliffcrest, Cliffside, Scarborough Village West</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>9</th>\n",
       "      <td>M1N</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Birch Cliff, Cliffside West</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>10</th>\n",
       "      <td>M1P</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Dorset Park, Scarborough Town Centre, Wexford ...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>11</th>\n",
       "      <td>M1R</td>\n",
       "      <td>Scarborough</td>\n",
       "      <td>Maryvale, Wexford</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   Postal Code      Borough                                       Neighborhood\n",
       "0          M1B  Scarborough                                     Rouge, Malvern\n",
       "1          M1C  Scarborough             Highland Creek, Rouge Hill, Port Union\n",
       "2          M1E  Scarborough                  Guildwood, Morningside, West Hill\n",
       "3          M1G  Scarborough                                             Woburn\n",
       "4          M1H  Scarborough                                          Cedarbrae\n",
       "5          M1J  Scarborough                                Scarborough Village\n",
       "6          M1K  Scarborough        East Birchmount Park, Ionview, Kennedy Park\n",
       "7          M1L  Scarborough                    Clairlea, Golden Mile, Oakridge\n",
       "8          M1M  Scarborough    Cliffcrest, Cliffside, Scarborough Village West\n",
       "9          M1N  Scarborough                        Birch Cliff, Cliffside West\n",
       "10         M1P  Scarborough  Dorset Park, Scarborough Town Centre, Wexford ...\n",
       "11         M1R  Scarborough                                  Maryvale, Wexford"
      ]
     },
     "execution_count": 6,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# combining the neighborhoods into one line separated by a comma.\n",
    "df_codes = df_codes.groupby(['Postal Code', 'Borough'])['Neighborhood'].apply(', '.join).reset_index()\n",
    "df_codes.columns = ['Postal Code', 'Borough', 'Neighborhood']\n",
    "df_codes.head(12)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(103, 3)"
      ]
     },
     "execution_count": 8,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "#last step is to use the .shape function\n",
    "df_codes.shape"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.7.0"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}
