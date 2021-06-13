## Implementation
    
We implemented an importer and exporter interface for the HAZOP2RDF project in Python. Using a command line 
interface users can interact with our software. 
     
Using our Importer API the user can import and validate incoming HAZOP data in Excel format and generate RDF 
graphs from it. They can be locally stored or uploaded to a Fuseki server. 
     
Using our Exporter API the user can export RDF Graphs containing HAZOP data to Excel format. The source for the 
Exporter API can either be a locally stored RDF file or an RDF file stored on a Fuseki server.
    
### Importer API
    
The main purpose of the Importer API is to build an RDF Graph from incoming Excel data. In order to build an 
RDF graph, we carefully read the incoming HAZOP Data and validate it. In order to validate correctly we 
implemented a config file, which stores all the metadata needed to describe the importing and validating process.
     
The main command of the Importer API is cmd-build-hazop-graphs, which reads the HAZOP data stored in a local 
directory and transforms it to an RDF Graph. The graph can be consequentially stored locally or uploaded to a 
Fuseki server. The two other commands cmd-list-excel-data and cmd-read-hazop-data make it possible for the user to 
perform steps of the main importer command individually.
     
The installation of a Fuseki server is optional. If the server is offline, the files can not be uploaded to the 
server resulting in an error message which is displayed to the user.
     
### Sequence Diagram: Importer API

* Diagram
    
### Exporter API
    
After the HAZOP data was successfully imported and stored, the user can convert the RDF Graph to Excel format again. 
     
There are two main commands in the Exporter API for the users to interact with. The user can either export data from 
an RDF file located in a local directory or from a file located online on a Fuseki server. In order for the export 
from Fuseki Server to function the server needs to be running and RDF Graphs need to be available. 
     
As a result the RDF Graphs will be stored locally in Excel format again.
     
### Sequence Diagram: Exporter API

* Diagram
    
### Remarks
    
We developed the HAZOP2RDF Project with version control on the online platform GitHub. The program is available for 
Windows and macOS platforms. We also included a detailed installation guide in the documentation on the GitHub platform.

```plantuml
skinparam monochrome true
skinparam ranksep 80
skinparam dpi 150
skinparam arrowThickness 0.7
skinparam packageTitleAlignment left
skinparam usecaseBorderThickness 0.4
skinparam defaultFontSize 16
skinparam rectangleBorderThickness 1
skinparam componentStyle rectangle

component "Command Line Interface" as CLI
component "Importer" as ImporterInterface << Interface >>
component "Exporter" as ExporterInterface << Interface >>
component "Importer" as ImporterService << Service >>
component "Exporter" as ExporterService << Service >>
component "TripleStore" as TripleStoreService << Service >>

CLI --> ImporterInterface
CLI --> ExporterInterface
ImporterInterface ..> ImporterService
ImporterInterface ..> TripleStoreService
ExporterInterface ..> ExporterService
ExporterInterface ..> TripleStoreService
```

```plantuml
actor User

participant "read-excel-data" as ReadExcelData << command >>
participant "read-hazop-data" as ReadHazopData << command >>
participant "build-hazop-graphs" as BuildHazopGraphs << command >>

database "localhost" as Localhost << storage >>
database "Fuseki server" as Fuseki << database >>


User -> ReadExcelData: << call >>
activate ReadExcelData

ReadExcelData --> User: excel data list
deactivate ReadExcelData

User -> ReadHazopData: << call >>
activate ReadHazopData

ReadHazopData -> ReadExcelData: << call >>
activate ReadExcelData

ReadExcelData --> User: excel data list
ReadExcelData --> ReadHazopData: return excel data list
deactivate ReadExcelData

ReadHazopData --> User: hazop data list
deactivate ReadHazopData

User -> BuildHazopGraphs: << call >>
activate BuildHazopGraphs

BuildHazopGraphs -> ReadHazopData: << call >>
activate ReadHazopData

ReadHazopData -> ReadExcelData: << call >>
activate ReadExcelData

ReadExcelData --> User: excel data list
ReadExcelData --> ReadHazopData: return excel data list
deactivate ReadExcelData

ReadHazopData --> User: hazop data list
ReadHazopData --> BuildHazopGraphs: return hazop data list
deactivate ReadHazopData

BuildHazopGraphs -> Localhost: save hazop data
BuildHazopGraphs -> Fuseki: upload hazop data
deactivate BuildHazopGraphs
activate Fuseki

Fuseki --> User: operation resposnse
deactivate Fuseki
```

```plantuml
actor User

participant "export-graphs-from\n-local-directory" as ExportFromStorage << command >>
participant "export-graphs-from\n-fuseki-server" as ExportFromFuseki << command >>

database "localhost" as Localhost << storage >>
database "Fuseki server" as Fuseki << database >>


User -> ExportFromStorage: << call >>
activate ExportFromStorage

ExportFromStorage -> Localhost: get hazop graph
activate Localhost

Localhost -> ExportFromStorage: return hazop graph
note left: early return if no data
deactivate Localhost

ExportFromStorage -> Localhost: save hazop graph in excel
ExportFromStorage --> User: operation response
deactivate ExportFromStorage


User -> ExportFromFuseki: << call >>
activate ExportFromFuseki

ExportFromFuseki -> Fuseki: get hazop graph
activate Fuseki

Fuseki -> ExportFromFuseki: return hazop graph
note left: early return if no data
deactivate Fuseki

ExportFromFuseki -> Localhost: save hazop graph in excel
ExportFromFuseki --> User: operation response
deactivate ExportFromFuseki
```