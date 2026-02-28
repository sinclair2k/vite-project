# ISO 20022 eRepository XML Structure

The eRepository and EMF model can be downloaded from https://www.iso20022.org/iso20022-repository/e-repository

All objects are identified by a `xmi:id` attribute (opaque string, e.g. `_7Wb_QDqQEeWPhIzq1hUQvA`).
Cross-references between objects are expressed as **space-delimited lists of `xmi:id` values** in
referencing attributes. Every element carrying a `xmi:id` can be a reference target.

---

## Abstract Base Classes

These classes are never serialized as XML elements directly but contribute attributes and
references to every concrete class that inherits from them.

### `ModelEntity` (root abstract)
| Feature            | Type        | Bounds | Notes                                                 |
|--------------------|-------------|--------|-------------------------------------------------------|
| `objectIdentifier` | EString     | [0..1] | opaque `xmi:id`                                       |
| `nextVersions`     | ModelEntity | [0..*] | space-list of `xmi:id`; opposite of `previousVersion` |
| `previousVersion`  | ModelEntity | [0..1] | opposite of `nextVersions`                            |

### `RepositoryConcept` extends ModelEntity
Inherited by every named, definable, status-tracked thing.

| Feature              | Type               | Bounds | Default                  | Notes       |
|----------------------|--------------------|--------|--------------------------|-------------|
| `name`               | EString            | [1..1] | —                        |             |
| `definition`         | EString            | [0..1] | —                        |             |
| `example`            | EString            | [0..*] | —                        |             |
| `registrationStatus` | RegistrationStatus | [1..1] | Provisionally Registered |             |
| `removalDate`        | EDate              | [0..1] | —                        |             |
| `semanticMarkup`     | SemanticMarkup     | [0..*] | —                        | containment |
| `doclet`             | Doclet             | [0..*] | —                        | containment |
| `constraint`         | Constraint         | [0..*] | —                        | containment |

### `MultiplicityEntity` (abstract)
Mixed in by `Construct` (and transitively by all elements with cardinality).

| Feature     | Type           | Bounds | Notes                                    |
|-------------|----------------|--------|------------------------------------------|
| `minOccurs` | EIntegerObject | [0..1] | absent = 1 in practice                   |
| `maxOccurs` | EIntegerObject | [0..1] | absent = 1; -1 serialised as `unbounded` |

### `Construct` extends RepositoryConcept, MultiplicityEntity (abstract)
Base for all field-level constructs.

| Feature  | Type    | Bounds |
|----------|---------|--------|
| `xmlTag` | EString | [0..1] |

---

## Type Hierarchy (relevant classes only)

```
ModelEntity (abstract)
└── RepositoryConcept (abstract)
    ├── TopLevelDictionaryEntry (abstract)           dataDictionary children
    │   ├── MessageComponentType (abstract)
    │   │   ├── MessageElementContainer (abstract)
    │   │   │   ├── MessageComponent                 iso20022:MessageComponent
    │   │   │   └── ChoiceComponent                  iso20022:ChoiceComponent
    │   │   ├── ExternalSchema                       iso20022:ExternalSchema
    │   │   └── UserDefined                          iso20022:UserDefined
    │   ├── DataType (abstract)
    │   │   ├── String (abstract)
    │   │   │   ├── Text                             iso20022:Text
    │   │   │   ├── IdentifierSet                    iso20022:IdentifierSet
    │   │   │   └── CodeSet                          iso20022:CodeSet
    │   │   ├── Decimal (abstract)
    │   │   │   ├── Quantity                         iso20022:Quantity
    │   │   │   ├── Rate                             iso20022:Rate
    │   │   │   └── Amount                           iso20022:Amount
    │   │   ├── Boolean (abstract)
    │   │   │   └── Indicator                        iso20022:Indicator
    │   │   ├── Binary                               iso20022:Binary
    │   │   ├── SchemaType                           iso20022:SchemaType
    │   │   └── AbstractDateTimeConcept (abstract)
    │   │       ├── Date / DateTime / Time
    │   │       ├── Year / YearMonth / Month / MonthDay / Day
    │   │       └── Duration
    │   └── BusinessComponent                        iso20022:BusinessComponent
    ├── TopLevelCatalogueEntry (abstract)            businessProcessCatalogue children
    │   ├── BusinessArea                             iso20022:BusinessArea
    │   ├── MessageSet                               iso20022:MessageSet
    │   └── (MessageChoreography, BusinessTransaction, BusinessProcess, …)
    └── Constraint / SemanticMarkup / Doclet / …
Construct (abstract, extends RepositoryConcept + MultiplicityEntity)
└── MessageConstruct (abstract)
    ├── MessageBuildingBlock
    └── MessageElement (abstract)
        ├── MessageAttribute                         xsi:type="iso20022:MessageAttribute"
        └── MessageAssociationEnd                   xsi:type="iso20022:MessageAssociationEnd"
```

---

## Top-level Document Structure

```
<iso20022:Repository xmi:version="2.0"
    xmlns:xmi="http://www.omg.org/XMI"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:iso20022="urn:iso:std:iso:20022:2013:ecore"
    xmi:id="...">

  <dataDictionary xmi:id="...">
    <topLevelDictionaryEntry xsi:type="iso20022:*" .../>  <!-- repeated, ~23k entries -->
  </dataDictionary>

  <businessProcessCatalogue xmi:id="...">
    <topLevelCatalogueEntry xsi:type="iso20022:*" .../>   <!-- 199 entries: MessageSet + BusinessArea -->
    <messageDefinition ...>                               <!-- 3,121 entries -->
      <messageBuildingBlock .../>
      <messageDefinitionIdentifier ./>
      <constraint .../>
      <semanticMarkup .../>
    </messageDefinition>
  </businessProcessCatalogue>

</iso20022:Repository>
```

---

## `<dataDictionary>` — Reusable Types and Components

Contains `<topLevelDictionaryEntry>` elements distinguished by `xsi:type`.

### Structural types

#### `iso20022:MessageComponent` (13,810 entries)
A reusable message-layer data structure. Referenced by `messageBuildingBlock.complexType` and
`messageElement[MessageAssociationEnd].type`.

```xml
<topLevelDictionaryEntry xsi:type="iso20022:MessageComponent"
    xmi:id="..."
    name="ATMAccountStatement1"
    definition="..."
    registrationStatus="Registered"
    trace="..."              <!-- → BusinessComponent.xmi:id -->
    nextVersions="..."       <!-- → MessageComponent.xmi:id (space list) -->
    previousVersion="...">   <!-- → MessageComponent.xmi:id -->
  <messageElement .../>      <!-- repeated -->
  <xors .../>                <!-- optional mutual-exclusion rules -->
  <constraint .../>
  <semanticMarkup .../>
</topLevelDictionaryEntry>
```

#### `iso20022:ChoiceComponent` (4,136 entries)
A choice group (XML `xs:choice` equivalent). Same structure and references as MessageComponent.

#### `iso20022:BusinessComponent` (785 entries)
Business-layer concept (not directly used in XML messages). Referenced by MessageComponent via `trace`.

```xml
<topLevelDictionaryEntry xsi:type="iso20022:BusinessComponent"
    xmi:id="..."
    name="ATMTotal"
    definition="..."
    registrationStatus="Registered"
    superType="..."            <!-- → BusinessComponent.xmi:id -->
    subType="..."              <!-- → BusinessComponent.xmi:id (space list) -->
    associationDomain="..."    <!-- → element[BusinessAssociationEnd].xmi:id -->
    derivationComponent="..."  <!-- → BusinessComponent.xmi:id (space list) -->
    derivationElement="...">   <!-- → element.xmi:id -->
  <element xsi:type="iso20022:BusinessAttribute" .../>
  <element xsi:type="iso20022:BusinessAssociationEnd" .../>
</topLevelDictionaryEntry>
```

#### `iso20022:ExternalSchema`
References an external XML schema namespace. Extends `MessageComponentType` (not DataType).

| Attribute        | Type           | Notes                       |
|------------------|----------------|-----------------------------|
| `namespaceList`  | EString [0..*] |                             |
| `processContent` | ProcessContent | `lax` \| `skip` \| `strict` |

#### `iso20022:UserDefined`
Wildcard / any-type placeholder. Extends `MessageComponentType`.

| Attribute         | Type           | Notes                          |
|-------------------|----------------|--------------------------------|
| `namespace`       | Namespace      | `##any` \| `##other` \| `list` |
| `namespaceList`   | EString [0..1] | used when namespace = `list`   |
| `processContents` | ProcessContent | `lax` \| `skip` \| `strict`    |

---

### Simple / primitive datatypes

All are `<topLevelDictionaryEntry>` with a datatype `xsi:type`. Referenced by `simpleType` attributes
on `messageElement[MessageAttribute]`, `messageBuildingBlock`, and `element[BusinessAttribute]`.

**DataType inheritance and attributes:**

| `xsi:type`                |   Count | Parent class            | Key attributes                                                                                                                     |
|---------------------------|--------:|-------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| `iso20022:Text`           |     145 | String                  | `pattern`, `minLength`, `maxLength`, `length`                                                                                      |
| `iso20022:IdentifierSet`  |      71 | String                  | `pattern`, `minLength`, `maxLength`, `identificationScheme`                                                                        |
| `iso20022:CodeSet`        |   3,973 | String                  | `pattern`, `minLength`, `maxLength`, `identificationScheme`; contains `<code>` children; `trace`/`derivation` for CodeSet ancestry |
| `iso20022:Quantity`       |      25 | Decimal                 | `minInclusive`, `maxInclusive`, `totalDigits`, `fractionDigits`, `unitCode`                                                        |
| `iso20022:Rate`           |       8 | Decimal                 | `totalDigits`, `fractionDigits`, `baseValue`, `baseUnitCode`                                                                       |
| `iso20022:Amount`         |      17 | Decimal                 | `totalDigits`, `fractionDigits`, `minInclusive`, `currencyIdentifierSet` (→ DataType.xmi:id)                                       |
| `iso20022:Indicator`      |      13 | Boolean                 | `meaningWhenTrue`, `meaningWhenFalse`, `pattern`                                                                                   |
| `iso20022:Binary`         |      14 | DataType                | `minLength`, `maxLength`, `length`, `pattern`                                                                                      |
| `iso20022:SchemaType`     |      44 | DataType                | `kind` (W3C built-in type name, e.g. `decimal`, `dateTime`)                                                                        |
| `iso20022:Date`           |       1 | AbstractDateTimeConcept | `minInclusive`, `maxInclusive`, `pattern`                                                                                          |
| `iso20022:DateTime`       |       2 | AbstractDateTimeConcept | same                                                                                                                               |
| `iso20022:Time`           |       1 | AbstractDateTimeConcept | same                                                                                                                               |
| `iso20022:Year`           |       2 | AbstractDateTimeConcept | same                                                                                                                               |
| `iso20022:YearMonth`      |       1 | AbstractDateTimeConcept | same                                                                                                                               |
| `iso20022:Month`          |       1 | AbstractDateTimeConcept | same                                                                                                                               |
| `iso20022:UserDefined`    |       2 | MessageComponentType    | `namespace`, `processContents`                                                                                                     |
| `iso20022:ExternalSchema` |       7 | MessageComponentType    | `namespaceList`, `processContent`                                                                                                  |

**String (abstract) attributes** (inherited by Text, IdentifierSet, CodeSet):
`minLength`, `maxLength`, `length`, `pattern` — all [0..1] EIntegerObject / EString.

**Decimal (abstract) attributes** (inherited by Quantity, Rate, Amount):
`minInclusive`, `minExclusive`, `maxInclusive`, `maxExclusive` [0..1] EString;
`totalDigits`, `fractionDigits` [0..1] EIntegerObject; `pattern` [0..1] EString.

**AbstractDateTimeConcept (abstract) attributes** (inherited by all date/time types):
`minInclusive`, `minExclusive`, `maxInclusive`, `maxExclusive`, `pattern` — all [0..1] EString.

#### `iso20022:CodeSet` in detail
```xml
<topLevelDictionaryEntry xsi:type="iso20022:CodeSet"
    xmi:id="..."  name="ActiveCurrencyCode"  definition="..."
    registrationStatus="Registered"
    trace="..."        <!-- → CodeSet.xmi:id (parent set this derives from) -->
    nextVersions="...">
  <code xmi:id="..." name="EUR" codeName="EUR" definition="..." registrationStatus="..."/>
</topLevelDictionaryEntry>
```

---

## `<businessProcessCatalogue>` — Messages and Business Areas

`businessProcessCatalogue` has two kinds of children:

```
businessProcessCatalogue
├── topLevelCatalogueEntry[MessageSet]    ← flat siblings, reference messages by ID
├── topLevelCatalogueEntry[MessageSet]
├── ...
├── topLevelCatalogueEntry[BusinessArea]  ← physically owns messageDefinition children
│     ├── messageDefinition
│     ├── messageDefinition
│     └── ...
└── topLevelCatalogueEntry[BusinessArea]
      └── ...
```

There is **no direct attribute** linking a `MessageSet` to a `BusinessArea` or vice versa.
They are connected only through `messageDefinition`: each `messageDefinition` is a physical child
of one `BusinessArea` and is also referenced by ID from one or more `MessageSet`s.

### `iso20022:BusinessArea` (36 entries)
Organizational domain that physically contains its `messageDefinition` elements.

```xml
<topLevelCatalogueEntry xsi:type="iso20022:BusinessArea"
    xmi:id="..."  name="ATM Card Transaction"
    definition="..."
    registrationStatus="Provisionally Registered"
    code="catp">       <!-- short code matching messageDefinitionIdentifier.businessArea -->
  <messageDefinition .../>   <!-- repeated — physically owned -->
</topLevelCatalogueEntry>
```

### `iso20022:MessageSet` (163 entries)
A named publication bundle. References messages by ID; does **not** contain them as children.

```xml
<topLevelCatalogueEntry xsi:type="iso20022:MessageSet"
    xmi:id="..."
    name="Account Switching - ISO - Latest version"
    definition="..."
    registrationStatus="Registered"
    messageDefinition="ID1 ID2 ID3 ...">  <!-- → messageDefinition.xmi:id (space list) -->
  <doclet type="docInfoTitle" content="..."/>   <!-- optional publication metadata -->
</topLevelCatalogueEntry>
```

**Note:** Multiple `MessageSet` entries share the same canonical name with suffixes
"ISO - Latest version", "ISO - Previous version", "ISO - Archive" indicating lifecycle stage.
A `messageDefinition` can belong to multiple `MessageSet`s (its `messageSet` attribute is also a
space list) but has exactly one `BusinessArea` parent.

### `<messageDefinition>` (3,121 entries)
Defines one ISO 20022 message. Extends `RepositoryType` → `RepositoryConcept`.

```xml
<messageDefinition
    xmi:id="..."
    name="ATMCompletionAcknowledgementV01"
    definition="..."
    registrationStatus="Registered"
    messageSet="MSID1 MSID2"   <!-- → MessageSet.xmi:id (space list, inverse of above) -->
    xmlTag="ATMCmpltnAck"
    xmlName="..."               <!-- alternative XML name, rarely present -->
    rootElement="Document"      <!-- default "Document"; the wrapper element in actual XML -->
    nextVersions="..."          <!-- → messageDefinition.xmi:id -->
    previousVersion="...">      <!-- → messageDefinition.xmi:id -->

  <messageBuildingBlock .../>   <!-- repeated, the top-level XML elements -->
  <messageDefinitionIdentifier businessArea="catp" messageFunctionality="009"
                               flavour="001" version="01"/>
  <constraint .../>
  <semanticMarkup .../>
  <xors .../>                   <!-- mutual-exclusion rules across building blocks -->
</messageDefinition>
```

#### `<messageBuildingBlock>`
Top-level fields of a message. Each maps to an XML element directly under `<Document>`.
Extends `MessageConstruct` → `Construct` (has `xmlTag`) + `MultiplicityEntity` (has `minOccurs`/`maxOccurs`) + `RepositoryConcept` (has `name`, `definition`, etc.).

```xml
<messageBuildingBlock
    xmi:id="..."
    name="Header"
    definition="..."
    registrationStatus="Provisionally Registered"
    xmlTag="Hdr"
    minOccurs="1"  maxOccurs="1"
    complexType="..."    <!-- → MessageComponent.xmi:id or ChoiceComponent.xmi:id -->
    simpleType="..."/>   <!-- → DataType.xmi:id (mutually exclusive with complexType) -->
```

---

## Child Elements of MessageComponent / ChoiceComponent

Both element types extend `MessageElement` → `MessageConstruct` → `Construct` + `MultiplicityEntity` + `RepositoryConcept`.

### `<messageElement xsi:type="iso20022:MessageAttribute">` (60,644)
A field with a primitive or complex type.

```xml
<messageElement xsi:type="iso20022:MessageAttribute"
    xmi:id="..."  name="AccountName"  definition="..."
    registrationStatus="Provisionally Registered"
    xmlTag="AcctNm"
    minOccurs="0"  maxOccurs="1"
    isDerived="false"
    simpleType="..."            <!-- → DataType.xmi:id (CodeSet, Text, etc.) -->
    complexType="..."           <!-- → MessageComponentType.xmi:id (rare, alternative to simpleType) -->
    businessElementTrace="..."  <!-- → element[BusinessAttribute].xmi:id -->
    nextVersions="..."/>
```

`simpleType` and `complexType` are mutually exclusive in practice; both are [0..1] in the metamodel.

### `<messageElement xsi:type="iso20022:MessageAssociationEnd">` (52,718)
A nested complex field referencing another component.

```xml
<messageElement xsi:type="iso20022:MessageAssociationEnd"
    xmi:id="..."  name="AccountIdentifier"  definition="..."
    registrationStatus="Provisionally Registered"
    xmlTag="AcctIdr"
    minOccurs="1"  maxOccurs="1"
    isDerived="false"  isComposite="true"    <!-- isComposite defaults to true -->
    type="..."                  <!-- → MessageComponent.xmi:id or ChoiceComponent.xmi:id (required) -->
    businessElementTrace="..."  <!-- → element[BusinessAssociationEnd].xmi:id -->
    businessComponentTrace="..." <!-- → BusinessComponent.xmi:id -->
    nextVersions="..."/>
```

`type` is [1..1] (required). `isComposite` defaults to `true`.

### `<xors>` — Mutual exclusion constraint
Appears in `MessageComponent`, `ChoiceComponent`, and `messageDefinition`.

```xml
<xors xmi:id="..."
    name="PreviousReferenceOrOtherReferenceRule"
    definition="Either PreviousReference or OtherReference may be present, but not both."
    registrationStatus="Provisionally Registered"
    impactedElements="ID1 ID2"/>  <!-- → messageElement.xmi:id (space list) -->
```

---

## Child Elements of BusinessComponent

### `<element xsi:type="iso20022:BusinessAttribute">` (2,371)
```xml
<element xsi:type="iso20022:BusinessAttribute"
    xmi:id="..."  name="ATMBalance"  definition="..."
    registrationStatus="Registered"
    minOccurs="1"  maxOccurs="1"
    isDerived="false"
    simpleType="..."      <!-- → DataType.xmi:id -->
    complexType="..."     <!-- → BusinessComponent.xmi:id (rare) -->
    derivation="ID1 ID2"  <!-- → messageElement[MessageAttribute].xmi:id (space list) -->
/>
```

### `<element xsi:type="iso20022:BusinessAssociationEnd">` (2,230)
```xml
<element xsi:type="iso20022:BusinessAssociationEnd"
    xmi:id="..."  name="RelatedCardPayment"  definition="..."
    registrationStatus="Registered"
    minOccurs="0"
    isDerived="false"
    type="..."       <!-- → BusinessComponent.xmi:id -->
    opposite="..."   <!-- → element[BusinessAssociationEnd].xmi:id (inverse end) -->
    derivation="..."  <!-- → messageElement[MessageAssociationEnd].xmi:id (space list) -->
/>
```

---

## ID Reference Summary

| Attribute                | On element                                                                        | Points to                                             |
|--------------------------|-----------------------------------------------------------------------------------|-------------------------------------------------------|
| `messageDefinition`      | `MessageSet`                                                                      | `messageDefinition.xmi:id` (space list)               |
| `messageSet`             | `messageDefinition`                                                               | `MessageSet.xmi:id` (space list)                      |
| `complexType`            | `messageBuildingBlock`, `messageElement[AssociationEnd]`                          | `MessageComponent.xmi:id` or `ChoiceComponent.xmi:id` |
| `complexType`            | `messageElement[Attribute]`, `element[BusinessAttribute]`                         | `MessageComponentType.xmi:id` (rare)                  |
| `simpleType`             | `messageBuildingBlock`, `messageElement[Attribute]`, `element[BusinessAttribute]` | DataType `xmi:id`                                     |
| `type`                   | `messageElement[AssociationEnd]`                                                  | `MessageComponent.xmi:id` or `ChoiceComponent.xmi:id` |
| `type`                   | `element[BusinessAssociationEnd]`                                                 | `BusinessComponent.xmi:id`                            |
| `trace`                  | `MessageComponent`, `ChoiceComponent`                                             | `BusinessComponent.xmi:id`                            |
| `trace`                  | `CodeSet`                                                                         | parent `CodeSet.xmi:id`                               |
| `businessElementTrace`   | `messageElement[Attribute]`                                                       | `element[BusinessAttribute].xmi:id`                   |
| `businessElementTrace`   | `messageElement[AssociationEnd]`                                                  | `element[BusinessAssociationEnd].xmi:id`              |
| `businessComponentTrace` | `messageElement[AssociationEnd]`                                                  | `BusinessComponent.xmi:id`                            |
| `opposite`               | `element[BusinessAssociationEnd]`                                                 | `element[BusinessAssociationEnd].xmi:id`              |
| `associationDomain`      | `BusinessComponent`                                                               | `element[BusinessAssociationEnd].xmi:id`              |
| `superType` / `subType`  | `BusinessComponent`                                                               | `BusinessComponent.xmi:id`                            |
| `currencyIdentifierSet`  | `Amount` datatype                                                                 | `DataType.xmi:id` (typically an `IdentifierSet`)      |
| `impactedElements`       | `xors`                                                                            | `messageElement.xmi:id` (space list)                  |
| `derivation`             | `element[BusinessAttribute\|AssociationEnd]`                                      | message element `xmi:id` (space list)                 |
| `derivationComponent`    | `BusinessComponent`                                                               | `BusinessComponent.xmi:id` (space list)               |
| `derivationElement`      | `BusinessComponent`                                                               | `element.xmi:id`                                      |
| `nextVersions`           | any                                                                               | same type `xmi:id` (space list)                       |
| `previousVersion`        | any                                                                               | same type `xmi:id`                                    |

---

## Registration Statuses

| Value                      |   Enum ordinal | Meaning                        |
|----------------------------|---------------:|--------------------------------|
| `Provisionally Registered` |    0 (default) | Draft / pending final approval |
| `Registered`               |              1 | Officially published           |
| `Obsolete`                 |              2 | Superseded, kept for reference |

---

## Miscellaneous Child Elements

| Element                         | Parent                                                     | Purpose                                                                                                              |
|---------------------------------|------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| `<messageDefinitionIdentifier>` | `messageDefinition`                                        | Structured identifier: `businessArea`, `messageFunctionality`, `flavour`, `version` (no `xmi:id`)                    |
| `<code>`                        | `CodeSet`                                                  | Enumeration value: `name`, `codeName`, `definition`                                                                  |
| `<constraint>`                  | any RepositoryConcept                                      | Rule: `name`, `definition`, `expression`, `expressionLanguage`                                                       |
| `<semanticMarkup>`              | any RepositoryConcept                                      | Metadata annotation: `type` (e.g. `Synonym`); contains `<elements>` with `name`/`value` key-value pairs              |
| `<doclet>`                      | any RepositoryConcept                                      | Publication metadata: `type`, `content`                                                                              |
| `<xors>`                        | `MessageComponent`, `ChoiceComponent`, `messageDefinition` | Mutual-exclusion rule between sibling `messageElement`s; `impactedElements` is space-list of `messageElement.xmi:id` |
| `<example>`                     | `IdentifierSet`                                            | Sample value text node                                                                                               |
