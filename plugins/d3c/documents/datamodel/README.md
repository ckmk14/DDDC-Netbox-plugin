# Datamodel within Netbox

Using Netbox in industrial control systems, the [Data Model of Netbox](https://netboxlabs.com/docs/netbox/en/stable/) has to be modified by custom field and plugins. Still, there are some major adjustments to be made for the device type in order to use it for matching the asset data base with the Common Security Advisory Framework [CSAF](link):question:.

## Datamodel for OT environment

With the [asset adminstration shell](https://webstore.iec.ch/publication/65628) there will be plenty of information about a device. However, which of those should be used for mapping vulnerabilities to a device? In the following table (industrialdigitaltwin.org)

*Table 1: Information for describing a device for vulnerability matching.*

| AAS | Netbox | CSAF | Description | Considered                     | Source |
|  -           |  - | -  |- | - | - |
| Manufacturer Name| [Manufacturer:Name](#manufacturer-name) | vendor | missing |Yes | [IDTA 02003 1 2](#idta-02003-1-2) |
| Manufacturer Part number| [DeviceType:part_number](#part-number) | sku |unique product identifier of the manufacturer, also called [article number](#idta-02006-2-0)| Yes |  [IDTA 02003 1 2](#idta-02003-1-2) |
| Manufacturer Order Code| DeviceType:order_code | N/A | By manufactures issued unique combination of numbers and letters used to identify the device for ordering. Not relevant for vulnerability matching| No | [IDTA 02003 1 2](#idta-02003-1-2) |
| Manufacturer Product Designation | [device?:Description? Designation](#device-description) | :question: | short description of the product, product group or function | sort of | [IDTA 02003 1 2](#idta-02003-1-2) |
| Nameplate| Manufacturer Product Root | N/A | Top level of a 3 level manufacturer specific product hierarchy | No | [IDTA 02006-2-0](#idta-02006-2-0)  |
| Nameplate| [DeviceType:family](#device-family) | product_family |2nd level of a 3 level manufacturer specific product hierarchy | yes | [IDTA 02006-2-0](#idta-02006-2-0) |
| Nameplate | [DeviceType:model](#model-number) | product_name | Characteristic to differentiate between different products of a product family or special variants| Yes | [IDTA 02006-2-0](#idta-02006-2-0) |
| Nameplate| [Device:serial_number](#serial-number) | serial_number |unique combination of numbers and letters used to identify the device once it has been manufactured. | Yes | [IDTA 02006-2-0](#idta-02006-2-0) |
| Nameplate| [Device:YoC](#year-of-construct) | N/A| year as completion date of object | Yes | [IDTA 02006-2-0](#idta-02006-2-0) |
| Date of Manufacture| N/A | N/A| Date from which the production and / or development process is completed or from which a service is provided completely |No | [IDTA 02006-2-0](#idta-02006-2-0) |
| URI | [URI](#x_generic_uris) | URI |Unique global identification of the product using a universal resource identifier (URI)| Yes | [IDTA 02007-1-0](#idta-02007-1-0) |
| SoftwareType | [SoftwareType](#software-type) | N/A |The type of the software (category, e.g. Runtime, Application, Firmeware, Driver, etc.) | Yes | [IDTA 02007-1-0](#idta-02007-1-0) |
| Version Name| [Software:version_name](#software-name) | N/A|The name this particular version is given | Yes | [IDTA 02007-1-0](#idta-02007-1-0) |
| Version | Software:version | *product_version and product_version_range* |The complete version information consisting of Major Version, Minor Version, Revision and Build Number | Yes | [IDTA 02007-1-0](#idta-02007-1-0) |
| Version Info| Version Info | N/A | Provides a textual description of most relevant characteristics of the version of the software | No | [IDTA 02007-1-0](#idta-02007-1-0) |
| Manufacturer Name | [Manufacturer Name](#manufacturer-name)  | As issuer or in product_tree, look up| in CSAF a link to SBOM is provided| Yes | [IDTA 02007-1-0](#idta-02007-1-0) |
| sure | Device:asset_tag :question: | N/A| Link to other company intern products like SAP | Yes | :question:|

### Literature

#### IDTA 02003-1-2

- **Title**: Generic Frame for Technical Data for Industrial Equipment in Manufacturing.
- **Section**: SMC GeneralInformation
- **Publisher**: Industrial digital twin Association
- **Year**: 2022
- **Source**: [IDTA Homepage](https://industrialdigitaltwin.org/content-hub/downloads)

#### IDTA 02006-2-0

- **Title**: Digital Nameplate for Industrial Equipment
- **Section**: Nameplate
- **Publisher**: Industrial digital twin Association
- **Year**: 2022
- **Source**: [IDTA Homepage](https://industrialdigitaltwin.org/content-hub/downloads)

#### IDTA 02007-1-0

- **Title**: Nameplate for Software in Manufacturing
- **Section**: SoftwareNameplateType
- **Publisher**: Industrial digital twin Association
- **Year**: 2023
- **Source**: [IDTA Homepage](https://industrialdigitaltwin.org/content-hub/downloads)

## Datamodel in Netbox

*Table 2: Datamodel for Netbox plugins by DINA community.*
|Name   | Netbox | Field | Desciption/Purpose | Example |
| - | - | - | - | - |
|Article Number         | Device:Interface.mac_address | :hammer: replace by part_number:hammer:| see [article number](#article-number---outdated) delete :exclamation: | -|
|Device role (primary)  | DeviceRole:name          | core| used for characerization and future featrues |see [Device Roles](https://netboxlabs.com/docs/netbox/en/stable/models/dcim/devicerole/) |
|Device role (secondary)| DeviceRole:name_minor    | custom| used for characerization and future featrues |see [Device Roles](https://netboxlabs.com/docs/netbox/en/stable/models/dcim/devicerole/) |
|Manufacturer           | Manufacturer:name        | core| manufacturer **of device type**| ABB, Schneider Electric|
|Device Type Name       | manufacturer + model     | N/A| used for assign a device to a device type.| [Details](#device-family) |
|Device Family          | DeviceType:device_family | custom /:exclamation:[Device Type](#discussion-device-type) | usually family a model is assigned to | [DeviceType](https://netboxlabs.com/docs/netbox/en/stable/models/dcim/devicetype/)|
|Model_number           | DeviceType:model  | core/:exclamation:[Device Type](#discussion-device-type) | Model number given by the manufacturer. Specification of a device_family | 6RA8096-4MV62-0AA0 [Details](#model-number) |
|SKU                    | DeviceType:part_number   | core/:exclamation:[Device Type](#discussion-device-type) | SKU (stock keeping unit) | [Details](#part-number) |
|Device Description     | DeviceType:device_description | custom/:exclamation:[Device Type](#discussion-device-type) | additional, optional field for detailed device description| [device description](#device-description)|
|Serial  Number         | Device:serial            | core | specific serial number of device | -|
|Communication partner - IP| Communication:dst_ip_addr | core | not observed CP but expected one (truth of state) for IDS | - |
|Communication partner - Protocol| Communication:transport_protocol| core | not observed CP but expected one (truth of state) |-|
|Hardware Name           |DeviceType:hardware_name  | :hammer:modify/custom/:exclamation:[Device Type](#discussion-device-type)  |FW of device, not of installed software (flag must be set in Netbox) | -|
|Hardware version        |DeviceType:hardware_version | :hammer:new/custom/:exclamation:[Device Type](#discussion-device-type) | Hardware version of the product; use "N/A" if just one version was build; use "unknown" if not known. The complete version information consisting of Major Version, Minor Version, Revision and Build Number | 4.0 --> filled to 4.0.0.0 |
|Software Manufacturer    |Software:manufacturer | :hammer: new :hammer: | destinguish between manufacturer of the device | |
|Firmware Name           |Software:name       | custom  |FW of device, not of installed software (flag must be set in Netbox) | -|
|Firmware Version        |Software:version    | :hammer: modify :hammer:  | FW of device, not of installed software (flag must be set in Netbox). The complete version information consisting of Major Version, Minor Version, Revision and Build Number | 4.0 --> filled to 4.0.0.0 |
|Software Name           |Software:name       | custom   |The name this particular version is given| - |
|Software Version        |Software:version    | :hammer: modify :hammer: | The complete version information consisting of Major Version, Minor Version, Revision and Build Number | 4.0 --> filled to 4.0.0.0 |
|Safety                  |Device:safety      | custom | device is used for safety functionality. Information also in CVSS available. |-|
|Exposure                |Device:Exposure    | custom | exposure to other networks| see [exposure](#exposure)|
|CPE                     |DeviceType:cpe     | custom | CSAF product identification helper|[CSAF-Standard 2.0](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html#31331-full-product-name-type---product-identification-helper---cpe)|
|Hashes                  |Software:Hashes    | custom | for firmare and applications software |[CSAF-Standard 2.0](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html#31332-full-product-name-type---product-identification-helper---hashes)|
|PURL                    |Software:Hashes    | custom | CSAF produ product identification helper | [CSAF-Standard 2.0](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html#31334-full-product-name-type---product-identification-helper---purl)|
|SBOM_URLs               |Software:sbom_urls | custom | The URL is a unique identifier. The content is secondary| [CSAF-Standard 2.0](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html#31335-full-product-name-type---product-identification-helper---sbom-urls)|
|x_generic_uris          |Software:x_generic_uris AND DeviceType:x_generic_uris| custom |unique name given by the vendor (e.g. [#649](https://github.com/oasis-tcs/csaf/issues/649))| [CSAF-Standard 2.0](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html#31338-full-product-name-type---product-identification-helper---generic-uris)|
|Age                     |Device:year     | :hammer: new :hammer: | Year of construction of the device| 2018|
|Inventory Number | Device:Inventar_number | :hammer: new :hammer: |  Not relevant for vulnerability matching. However, for linking the dataset to other interal products like SAP | - |

## Discussion Device Type

The problem with the core field in Netbox for Device Type is that the model is unique and also the full identification for the device type. This leads to problems, when describing a device with [CSAF](link), since there is more than just a model (name) such as product family, product name and a stock keeping unit (sku) as illustrated with the following example:

| attribute | Netbox | DDDC |
|:---:|:----:|:---:|
| manufacturer | Rockwell Automation| Rockwell Automation|
| family | N/A | ControllLogix |
| model (number) | ControlLogix Rack K - 10 Slot| Rack K -10 Slot|
| part_number | 1756-A10K | 1756-A10K (sku)|
| | | |

This issue is addressed in a proposal for the [Data model device Type #14125](https://github.com/netbox-community/netbox/discussions/14125).

### Model Name

The model name is not important for mapping assets to CSAF documents, but to assign devices correctly in Netbox. Therefore, the proposal for the [Data model device Type #14125](https://github.com/netbox-community/netbox/discussions/14125) was made. It should be the sum of

- [device family](#device-family)
- [model number](#model-number)
- [part number](#part-number)
- [hardware version](#hardware-version)

### Device Family

#### device family Netbox

Not available. It is a part of the model name.

#### device family DINA

Text field added to the Device Type object. Specifies the family of a model (device type) (e.g. SIMATIC, SCALANCE) is assigned to.

### model number

#### model number Netbox

In Netbox the name convention is a little bit misleading, since under [devicetype-library](https://github.com/netbox-community/devicetype-library) a `model` is defined as :

```text
The model number of the device type. This must be unique per manufacturer.
```

So the object `model` is the model name as well as model number.

#### model number DINA

A model number can be used as an article number. However, an article number is not always/necessarily a model number. Usually, all products have model numbers, often they are listed on the sticker on the device besides the serial number

#### model number CSAF

```text
The terms "model", "model number" and "model variant" are mostly used synonymously. Often it is abbreviated as "MN", M/N" or "model no.".
```

### part number

#### Netbox part number

```text
An alternative representation of the model number (e.g. a SKU). 
```

#### part number - CSAF

CSAF defines the product identification helper [SKU](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html#31337-full-product-name-type---product-identification-helper---skus):

```text
Any given stock keeping unit of value type string with at least 1 character represents a full or abbreviated (partial) stock keeping unit (SKU) of the component to identify. Sometimes this is also called "item number", "article number" or "product number".
```

#### part number - DINA

It can be the same as model_number, especially when seller is the vendor itself. This can be used as an alternative presentation of the model number [e.g. SKU as in devicetype-library](https://github.com/netbox-community/devicetype-library).

### Device Description

Text fields added to the Device Type object. Intended as an additional reminder alongside the device type name (e.g. CPU 414-3 PN/DP Zentralbaugruppe mit: Arbeitsspeicher 4 MB ...). Could be partially part of full_product_name_t/name in a CSAF document.

### Hardware Version

Text fields added to the Device Type object. Specifies the hardware version of the device type. Hardware version can be “N/A” if just one version was build. Multiple products exist in multiple hardware versions (due to PCB layout changes or chip shortages or hardware improvements), which can have impact on the software that can be used with the device.

## Further Description

### Article Number - outdated

Text fields added to the Device Type object. Specifies the stock keeping unit (SKU).  It can be the same as model number (NetBox: part\_number), especially when seller is the vendor itself.

### Communication partner - IP

not observed communication partner IP but expected one (truth of state)

### Communication partner - Protocol

not observed communication partner protocol but expected one (truth of state)

### CPE

Text fields added to the Device Type object. Specifies the Common Platform Enumeration (CPE) string of the device type.

### Device Status

Values are a specified enumeration already present in NetBox.

### Exposure

Selection added to the Device Type object. Specifies the grade of exposure to other networks. Valid values are:

- Small:  The asset is in a highly isolated and controlled zone. There are no connections from this cyber asset’s zone to or from a zone with lower trust.
- Indirect: The asset has no direct access to a zone with lower trust, but other cyber assets in this cyber asset’s zone are accessible to or from a zone with lower trust.
- Direct: The asset is directly accessible to or from a zone with lower trust.
- Unknown: Value if category for exposure is unknown.  

### Is Router

Selection added to the Interface object. Specifies whether one of the device's interfaces is a router interface or not. Valid values are:

- Yes: Yes, if it is a router interface.
- No: No, if it is not a router interface.
- Maybe: Maybe, if it might be a router interface.
- Unknown:  Value if category is unknown.

### Manufacturer Name

Legally valid designation of the natural or judicial body which is directly responsible for the design, production, packaging
and labeling of a product in respect to its being brought into the market.[IDTA 02003 1 2](#idta-02003-1-2)

In DINA it is used in the plugin under software and as core input of the device type in Netbox.

### Safety

Boolean field added to the Device object. Specifies, if the device is used/provides safety functionality.

### Secondary Roles

Multiple objects field added to the Device object. It should be possible to assign multiple Device Roles to a device. Therefore, this custom field enables the user to designate multiple Device Roles for a device using this feature. Additionally, the existing 'Role' attribute of a device should be understood as the primary role of the device.

### Serial Number

unique combination of numbers and letters used to identify the device once it has been manufactured. [IDTA 2006](#idta-02006-2-0)

Helps to determine the affectedness. For example, a batch (SN range) has been shipped with a FW that contains a vulnerability.

### Software Name

Examples: OS like Linux or libraries in python.

### Software Type

It has to be distinguished between firmware and additional software by the flag "is firmware".

### x_generic_uris

Unique name given by the vendor [example](https://github.com/oasis-tcs/csaf/issues/649). Hardware and software, can have one or more x_generic_uri. However, an x_generic_uri can only belong to one hardware resp. software.

### Year of Construct

This information might be relevant for legacy products when mapping against new information where the product is renamed or listed under a new vendor.