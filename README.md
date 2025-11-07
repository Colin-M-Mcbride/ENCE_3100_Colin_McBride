ENCE 3100 with Professor Goncalo Fernandes Pereira Martins

```verilog
classDiagram
    class DataViewerApp {
        -TemperatureDataManager dataManager
        -UserInterface ui
        -ViewSettings settings
        -PlotRenderer plotRenderer
        +main(String[] args)
        +run()
        -initialize()
    }

    class TemperatureDataManager {
        -List~TemperatureRecord~ allData
        -String currentCountry
        -List~String~ availableStates
        -int minYear
        -int maxYear
        +loadDataFile(String filename)
        +setCountry(String country)
        +getStatesForCountry() List~String~
        +getFilteredData(ViewSettings settings) List~TemperatureRecord~
        +getMinYear() int
        +getMaxYear() int
        +getAvailableCountries() List~String~
    }

    class TemperatureRecord {
        -String country
        -String state
        -int year
        -int month
        -double temperature
        +TemperatureRecord(String country, String state, int year, int month, double temp)
        +getCountry() String
        +getState() String
        +getYear() int
        +getMonth() int
        +getTemperature() double
    }

    class ViewSettings {
        -String selectedCountry
        -String selectedState
        -int startYear
        -int endYear
        -VisualizationType visualizationType
        +ViewSettings()
        +setCountry(String country)
        +setState(String state)
        +setStartYear(int year)
        +setEndYear(int year)
        +setVisualizationType(VisualizationType type)
        +getSelectedCountry() String
        +getSelectedState() String
        +getStartYear() int
        +getEndYear() int
        +getVisualizationType() VisualizationType
        +isValid() boolean
    }

    class VisualizationType {
        <<enumeration>>
        RAW
        EXTREMA
    }

    class UserInterface {
        -Scanner scanner
        -ViewSettings settings
        -TemperatureDataManager dataManager
        +UserInterface(TemperatureDataManager manager, ViewSettings settings)
        +displayMainMenu()
        +handleUserInput() MenuAction
        +promptForState() String
        +promptForCountry() String
        +promptForYear(String prompt, int min, int max) int
        +promptForVisualization() VisualizationType
        -displayCurrentSettings()
    }

    class MenuAction {
        <<enumeration>>
        CHANGE_STATE
        CHANGE_COUNTRY
        CHANGE_START_YEAR
        CHANGE_END_YEAR
        CHANGE_VISUALIZATION
        PLOT
        QUIT
        INVALID
    }

    class PlotRenderer {
        -ViewSettings settings
        -TemperatureColorMapper colorMapper
        +PlotRenderer(ViewSettings settings)
        +renderPlot(List~TemperatureRecord~ data)
        +displayPlotMenu()
        +handlePlotInput() PlotAction
        -renderRawView(List~TemperatureRecord~ data)
        -renderExtremaView(List~TemperatureRecord~ data)
        -calculateExtremaThresholds(List~TemperatureRecord~ data) Map
    }

    class PlotAction {
        <<enumeration>>
        RETURN_TO_MENU
        QUIT
    }

    class TemperatureColorMapper {
        +getColorForTemperature(double temp, double min, double max) String
        +getExtremaColor(double temp, double monthMin, double monthMax) String
        -isInExtremaRange(double temp, double min, double max) boolean
        -interpolateColor(double value, double min, double max) String
    }

    DataViewerApp "1" --> "1" TemperatureDataManager : uses
    DataViewerApp "1" --> "1" UserInterface : uses
    DataViewerApp "1" --> "1" ViewSettings : uses
    DataViewerApp "1" --> "1" PlotRenderer : uses
    
    TemperatureDataManager "1" --> "*" TemperatureRecord : manages
    
    UserInterface "1" --> "1" TemperatureDataManager : queries
    UserInterface "1" --> "1" ViewSettings : modifies
    UserInterface ..> MenuAction : returns
    
    ViewSettings "1" --> "1" VisualizationType : uses
    
    PlotRenderer "1" --> "1" ViewSettings : reads
    PlotRenderer "1" --> "1" TemperatureColorMapper : uses
    PlotRenderer ..> PlotAction : returns

```
