# Setup the Jupyter version of Dash
from jupyter_dash import JupyterDash

# Configure the necessary Python module imports for dashboard components
import dash_leaflet as dl
from dash import dcc, html
import plotly.express as px
from dash import dash_table
from dash.dependencies import Input, Output, State
import base64
JupyterDash.infer_jupyter_proxy_config()

# Configure OS routines
import os

# Configure the plotting routines
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt


#### FIX ME #####
# change animal_shelter and AnimalShelter to match your CRUD Python module file name and class name
from CRUD_Python_Module import CRUD

###########################
# Data Manipulation / Model
###########################
# FIX ME update with your username and password and CRUD Python module name

username = "aacuser"
password = "SNHU1234"

# Connect to database via CRUD Module
db = CRUD(username, password)

# class read method must support return of list object and accept projection json input
# sending the read method an empty document requests all documents be returned
df = pd.DataFrame.from_records(db.read({}))

# MongoDB v5+ is going to return the '_id' column and that is going to have an 
# invlaid object type of 'ObjectID' - which will cause the data_table to crash - so we remove
# it in the dataframe here. The df.drop command allows us to drop the column. If we do not set
# inplace=True - it will reeturn a new dataframe that does not contain the dropped column(s)
df.drop(columns=['_id'],inplace=True)

## Debug
# print(len(df.to_dict(orient='records')))
# print(df.columns)


#########################
# Dashboard Layout / View
#########################
app = JupyterDash(__name__)

# Grazioso Salvare Logo
image_filename = 'Grazioso Salvare Logo.png'
encoded_image = base64.b64encode(open(image_filename, 'rb').read())

#FIX ME Place the HTML image tag in the line below into the app.layout code according to your design
#FIX ME Also remember to include a unique identifier such as your name or date
#html.Img(src='data:image/png;base64,{}'.format(encoded_image.decode()))

app.layout = html.Div([
    html.Center([
        html.Img(
            src='data:image/png;base64,{}'.format(encoded_image.decode()),
            style={'height': '200px'}
        ),
        html.H1("Grazioso Salvare Dashboard"),
        html.H3("Yagazie Onuoha - CS 340 Project Two")
    ]),
    html.Hr(),
        
#FIXME Add in code for the interactive filtering options. For example, Radio buttons, drop down, checkboxes, etc.

    html.Div([
    dcc.RadioItems(
        id='filter-type',
        options=[
            {'label': 'Reset', 'value': 'RESET'},
            {'label': 'Water Rescue', 'value': 'WATER'},
            {'label': 'Mountain/Wilderness Rescue', 'value': 'MOUNTAIN'},
            {'label': 'Disaster/Individual Tracking', 'value': 'DISASTER'}
        ],
        value='RESET',
        labelStyle={'display': 'inline-block', 'margin-right': '20px'}
    )
]),
    html.Hr(),
    dash_table.DataTable(
    id='datatable-id',

    columns=[
        {
            "name": i,
            "id": i,
            "deletable": False,
            "selectable": True
        }
        for i in df.columns
    ],

    data=df.to_dict('records'),

    # User-friendly table features
    page_size=10,
    sort_action="native",
    filter_action="native",
    row_selectable="single",
    selected_rows=[0],

    style_table={
        'overflowX': 'auto'
    },

    style_cell={
        'textAlign': 'left'
    }
),
                        
    html.Br(),
    html.Hr(),
#This sets up the dashboard so that your chart and your geolocation chart are side-by-side
    html.Div(className='row',
         style={'display' : 'flex'},
             children=[
        html.Div(
            id='graph-id',
            className='col s12 m6',

            ),
        html.Div(
            id='map-id',
            className='col s12 m6',
            )
        ])
])

#############################################
# Interaction Between Components / Controller
#############################################



    
@app.callback(
    Output('datatable-id','data'),
    [Input('filter-type', 'value')]
)
def update_dashboard(filter_type):

    if filter_type == "WATER":

        query = {
            "breed": {
                "$in": [
                    "Labrador Retriever Mix",
                    "Chesapeake Bay Retriever",
                    "Newfoundland"
                ]
            },
            "sex_upon_outcome": "Intact Female",
            "age_upon_outcome_in_weeks": {
                "$gte": 26,
                "$lte": 156
            }
        }

    elif filter_type == "MOUNTAIN":

        query = {
            "breed": {
                "$in": [
                    "German Shepherd",
                    "Alaskan Malamute",
                    "Old English Sheepdog",
                    "Siberian Husky",
                    "Rottweiler"
                ]
            },
            "sex_upon_outcome": "Intact Male",
            "age_upon_outcome_in_weeks": {
                "$gte": 26,
                "$lte": 156
            }
        }

    elif filter_type == "DISASTER":

        query = {
            "breed": {
                "$in": [
                    "Doberman Pinscher",
                    "German Shepherd",
                    "Golden Retriever",
                    "Bloodhound",
                    "Rottweiler"
                ]
            },
            "sex_upon_outcome": "Intact Male",
            "age_upon_outcome_in_weeks": {
                "$gte": 20,
                "$lte": 300
            }
        }

    else:
        # Reset button
        query = {}


    # Get filtered data from MongoDB
    dff = pd.DataFrame.from_records(db.read(query))

    # Remove MongoDB ID
    if '_id' in dff.columns:
        dff.drop(columns=['_id'], inplace=True)

    return dff.to_dict('records')

# Display the breeds of animal based on quantity represented in
# the data table
@app.callback(
    Output('graph-id', "children"),
    [Input('datatable-id', "derived_virtual_data")]
)
def update_graphs(viewData):

    if viewData is None:
        dff = df
    else:
        dff = pd.DataFrame.from_dict(viewData)

    return [
        dcc.Graph(
            figure=px.pie(
                dff,
                names='breed',
                title='Preferred Rescue Dog Breeds'
            )
        )
    ]
    
#This callback will highlight a cell on the data table when the user selects it
@app.callback(
    Output('datatable-id', 'style_data_conditional'),
    [Input('datatable-id', 'selected_columns')]
)
def update_styles(selected_columns):
    return [{
        'if': { 'column_id': i },
        'background_color': '#D2F3FF'
    } for i in selected_columns]


# This callback will update the geo-location chart for the selected data entry
# derived_virtual_data will be the set of data available from the datatable in the form of 
# a dictionary.
# derived_virtual_selected_rows will be the selected row(s) in the table in the form of
# a list. For this application, we are only permitting single row selection so there is only
# one value in the list.
# The iloc method allows for a row, column notation to pull data from the datatable
@app.callback(
    Output('map-id', "children"),
    [Input('datatable-id', "derived_virtual_data"),
     Input('datatable-id', "derived_virtual_selected_rows")])
def update_map(viewData, index):

    if viewData is None:
        dff = df
    else:
        dff = pd.DataFrame.from_dict(viewData)

    if index is None or len(index) == 0:
        row = 0
    else:
        row = index[0]
        
    # Austin TX is at [30.75,-97.48]
    return [
        dl.Map(style={'width': '1000px', 'height': '500px'}, center=[30.75,-97.48], zoom=10, children=[
            dl.TileLayer(id="base-layer-id"),
            # Marker with tool tip and popup
            # Column 13 and 14 define the grid-coordinates for the map
            # Column 4 defines the breed for the animal
            # Column 9 defines the name of the animal
            dl.Marker(position=[dff.iloc[row,13],dff.iloc[row,14]], children=[
                dl.Tooltip(dff.iloc[row,4]),
                dl.Popup([
                    html.H1("Animal Name"),
                    html.P(dff.iloc[row,9])
                ])
            ])
        ])
    ]


# Run app and display result in jupyterlab mode, note, if you have previously run a prior app, the default port of 8050 may not be available, if so, try setting an alternate port.
app.run_server() 

Dash app running on https://bermudavega-japanlearn-3000.codio.io/proxy/8050/
