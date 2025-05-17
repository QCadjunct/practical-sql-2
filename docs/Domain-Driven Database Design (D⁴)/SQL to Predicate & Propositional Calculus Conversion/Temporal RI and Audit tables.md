# SQL to Predicate & Propositional Calculus Conversion

```sql
-- Predicate: Domain(Analytics) → Analytics ∈ Database
CREATE SCHEMA "Analytics"
-- Predicate: Domain(Audit) → Audit ∈ Database
CREATE SCHEMA "Audit"
-- Predicate: Domain(dBlob) → dBlob ∈ Database
CREATE SCHEMA "dBlob"
-- Predicate: Domain(DbSecurity) → DbSecurity ∈ Database
CREATE SCHEMA "DbSecurity"
-- Predicate: Domain(dDateTime) → dDateTime ∈ Database
CREATE SCHEMA "dDateTime"
-- Predicate: Domain(dEuropeanCarDealership) → dEuropeanCarDealership ∈ Database
CREATE SCHEMA "dEuropeanCarDealership"
-- Predicate: Domain(dNumber) → dNumber ∈ Database
CREATE SCHEMA "dNumber"
-- Predicate: Domain(dString) → dString ∈ Database
CREATE SCHEMA "dString"
-- Predicate: Domain(Hashing) → Hashing ∈ Database
CREATE SCHEMA "Hashing"
-- Predicate: Domain(HumanResources) → HumanResources ∈ Database
CREATE SCHEMA "HumanResources"
-- Predicate: Domain(Inventory) → Inventory ∈ Database
CREATE SCHEMA "Inventory"
-- Predicate: Domain(JsonOutput) → JsonOutput ∈ Database
CREATE SCHEMA "JsonOutput"
-- Predicate: Domain(LoadData) → LoadData ∈ Database
CREATE SCHEMA "LoadData"
-- Predicate: Domain(Output) → Output ∈ Database
CREATE SCHEMA "Output"
-- Predicate: Domain(PKSequence) → PKSequence ∈ Database
CREATE SCHEMA "PKSequence"
-- Predicate: Domain(Process) → Process ∈ Database
CREATE SCHEMA "Process"
-- Predicate: Domain(Sales) → Sales ∈ Database
CREATE SCHEMA "Sales"
-- Predicate: Domain(sdAddressesString) → sdAddressesString ∈ Database
CREATE SCHEMA "sdAddressesString"
-- Predicate: Domain(sdAudit) → sdAudit ∈ Database
CREATE SCHEMA "sdAudit"
-- Predicate: Domain(sdBusinessComponentString) → sdBusinessComponentString ∈ Database
CREATE SCHEMA "sdBusinessComponentString"
-- Predicate: Domain(sdCountryStringVariants) → sdCountryStringVariants ∈ Database
CREATE SCHEMA "sdCountryStringVariants"
-- Predicate: Domain(sdDate) → sdDate ∈ Database
CREATE SCHEMA "sdDate"
-- Predicate: Domain(sdFlagBit) → sdFlagBit ∈ Database
CREATE SCHEMA "sdFlagBit"
-- Predicate: Domain(sdFlagString) → sdFlagString ∈ Database
CREATE SCHEMA "sdFlagString"
-- Predicate: Domain(sdHashKey) → sdHashKey ∈ Database
CREATE SCHEMA "sdHashKey"
-- Predicate: Domain(sdHaskKey) → sdHaskKey ∈ Database
CREATE SCHEMA "sdHaskKey"
-- Predicate: Domain(sdLongTextString) → sdLongTextString ∈ Database
CREATE SCHEMA "sdLongTextString"
-- Predicate: Domain(sdMarketingTextString) → sdMarketingTextString ∈ Database
CREATE SCHEMA "sdMarketingTextString"
-- Predicate: Domain(sdOrdinalNumber) → sdOrdinalNumber ∈ Database
CREATE SCHEMA "sdOrdinalNumber"
-- Predicate: Domain(sdPersonNameString) → sdPersonNameString ∈ Database
CREATE SCHEMA "sdPersonNameString"
-- Predicate: Domain(sdProjectString) → sdProjectString ∈ Database
CREATE SCHEMA "sdProjectString"
-- Predicate: Domain(sdSequenceInteger) → sdSequenceInteger ∈ Database
CREATE SCHEMA "sdSequenceInteger"
-- Predicate: Domain(sdSequenceNumber) → sdSequenceNumber ∈ Database
CREATE SCHEMA "sdSequenceNumber"
-- Predicate: Domain(sdShortDescriptionString) → sdShortDescriptionString ∈ Database
CREATE SCHEMA "sdShortDescriptionString"
-- Predicate: Domain(sdShortTextString) → sdShortTextString ∈ Database
CREATE SCHEMA "sdShortTextString"
-- Predicate: Domain(sdSysTime) → sdSysTime ∈ Database
CREATE SCHEMA "sdSysTime"
-- Predicate: Domain(sdTimeString) → sdTimeString ∈ Database
CREATE SCHEMA "sdTimeString"
-- Predicate: Domain(sdVehicleDescriptorString) → sdVehicleDescriptorString ∈ Database
CREATE SCHEMA "sdVehicleDescriptorString"
-- Predicate: Domain(sdVehicleSalePayment) → sdVehicleSalePayment ∈ Database
CREATE SCHEMA "sdVehicleSalePayment"
-- Predicate: Domain(Triggered) → Triggered ∈ Database
CREATE SCHEMA "Triggered"
-- Predicate: Domain(Utils) → Utils ∈ Database
CREATE SCHEMA "Utils"

-- Predicate: TypeDomain(dbo.sdHashKey) → varbinary(32) ∪ {NULL}
    CREATE TYPE "dbo"."sdHashKey" FROM "varbinary"(32) NULL
-- Predicate: TypeDomain(dDateTime.sdDate) → datetime2(7) ∪ {NULL}
    CREATE TYPE "dDateTime"."sdDate" FROM "datetime2"(7) NULL
-- Predicate: TypeDomain(dDateTime.sdSysTime) → datetime2(7) ∧ ∀x ∈ dDateTime.sdSysTime: x ≠ NULL
    CREATE TYPE "dDateTime"."sdSysTime" FROM "datetime2"(7) NOT NULL
-- Predicate: TypeDomain(dEuropeanCarDealership.dDatetime) → datetime2(7) ∪ {NULL}
    CREATE TYPE "dEuropeanCarDealership"."dDatetime" FROM "datetime2"(7) NULL
-- Predicate: TypeDomain(dEuropeanCarDealership.dEuropeanCarManufacturer) → char(18) ∪ {NULL}
    CREATE TYPE "dEuropeanCarDealership"."dEuropeanCarManufacturer" FROM "char"(18) NULL
-- Predicate: TypeDomain(dEuropeanCarDealership.dNumber) → int ∪ {NULL}
    CREATE TYPE "dEuropeanCarDealership"."dNumber" FROM "int" NULL
-- Predicate: TypeDomain(dEuropeanCarDealership.dString) → char(18) ∪ {NULL}
    CREATE TYPE "dEuropeanCarDealership"."dString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dNumber.sdFlagBit) → int ∪ {NULL}
    CREATE TYPE "dNumber"."sdFlagBit" FROM "int" NULL
-- Predicate: TypeDomain(dNumber.sdOrdinalNumber) → int ∧ ∀x ∈ dNumber.sdOrdinalNumber: x ≠ NULL
    CREATE TYPE "dNumber"."sdOrdinalNumber" FROM "int" NOT NULL
-- Predicate: TypeDomain(dNumber.sdSequenceNumber) → int ∧ ∀x ∈ dNumber.sdSequenceNumber: x ≠ NULL
    CREATE TYPE "dNumber"."sdSequenceNumber" FROM "int" NOT NULL
-- Predicate: TypeDomain(dNumber.sdVehicleSalePayment) → money ∧ ∀x ∈ dNumber.sdVehicleSalePayment: x ≠ NULL
    CREATE TYPE "dNumber"."sdVehicleSalePayment" FROM "money" NOT NULL
-- Predicate: TypeDomain(dString.sdAddressesString) → varchar(60) ∪ {NULL}
    CREATE TYPE "dString"."sdAddressesString" FROM "varchar"(60) NULL
-- Predicate: TypeDomain(dString.sdBusinessComponentsString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdBusinessComponentsString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdCountryStringVariants) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdCountryStringVariants" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdCustomerString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdCustomerString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdFlagString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdFlagString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdLongTextString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdLongTextString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdPersonNameString) → char(18) ∧ ∀x ∈ dString.sdPersonNameString: x ≠ NULL
    CREATE TYPE "dString"."sdPersonNameString" FROM "char"(18) NOT NULL
-- Predicate: TypeDomain(dString.sdProjectString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdProjectString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdShortTextString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdShortTextString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdTimeString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdTimeString" FROM "char"(18) NULL
-- Predicate: TypeDomain(dString.sdVehicleString) → char(18) ∪ {NULL}
    CREATE TYPE "dString"."sdVehicleString" FROM "char"(18) NULL
-- Predicate: TypeDomain(sdAddressesString.AddressString) → char(60) ∪ {NULL}
    CREATE TYPE "sdAddressesString"."AddressString" FROM "char"(60) NULL
-- Predicate: TypeDomain(sdAddressesString.CountryString) → char(20) ∪ {NULL}
    CREATE TYPE "sdAddressesString"."CountryString" FROM "char"(20) NULL
-- Predicate: TypeDomain(sdAddressesString.PostalCodeString) → varchar(9) ∪ {NULL}
    CREATE TYPE "sdAddressesString"."PostalCodeString" FROM "varchar"(9) NULL
-- Predicate: TypeDomain(sdAddressesString.RegionString) → varchar(20) ∪ {NULL}
    CREATE TYPE "sdAddressesString"."RegionString" FROM "varchar"(20) NULL
-- Predicate: TypeDomain(sdAddressesString.TownString) → char(18) ∪ {NULL}
    CREATE TYPE "sdAddressesString"."TownString" FROM "char"(18) NULL
-- Predicate: TypeDomain(sdAudit.DbAction) → char(1) ∪ {NULL}
    CREATE TYPE "sdAudit"."DbAction" FROM "char"(1) NULL
-- Predicate: TypeDomain(sdBusinessComponentString.DepartmentString) → char(50) ∪ {NULL}
    CREATE TYPE "sdBusinessComponentString"."DepartmentString" FROM "char"(50) NULL
-- Predicate: TypeDomain(sdBusinessComponentString.ManufacturerVehicleMarketingType) → nvarchar(50) ∪ {NULL}
    CREATE TYPE "sdBusinessComponentString"."ManufacturerVehicleMarketingType" FROM "nvarchar"(50) NULL
-- Predicate: TypeDomain(sdCountryStringVariants.CountryISO2) → char(2) ∪ {NULL}
    CREATE TYPE "sdCountryStringVariants"."CountryISO2" FROM "char"(2) NULL
-- Predicate: TypeDomain(sdCountryStringVariants.CountryISO3) → char(3) ∪ {NULL}
    CREATE TYPE "sdCountryStringVariants"."CountryISO3" FROM "char"(3) NULL
-- Predicate: TypeDomain(sdDate.PurchaseDate) → datetime2(7) ∪ {NULL}
    CREATE TYPE "sdDate"."PurchaseDate" FROM "datetime2"(7) NULL
-- Predicate: TypeDomain(sdDate.SaleDate) → datetime2(7) ∪ {NULL}
    CREATE TYPE "sdDate"."SaleDate" FROM "datetime2"(7) NULL
-- Predicate: TypeDomain(sdFlagBit.FlagFalse) → int ∪ {NULL}
    CREATE TYPE "sdFlagBit"."FlagFalse" FROM "int" NULL
-- Predicate: TypeDomain(sdFlagString.AuditFlag) → char(1) ∪ {NULL}
    CREATE TYPE "sdFlagString"."AuditFlag" FROM "char"(1) NULL
-- Predicate: TypeDomain(sdFlagString.FlagYesNoString) → char(1) ∪ {NULL}
    CREATE TYPE "sdFlagString"."FlagYesNoString" FROM "char"(1) NULL
-- Predicate: TypeDomain(sdHashKey.RowLevelHashKey) → varbinary(32) ∪ {NULL}
    CREATE TYPE "sdHashKey"."RowLevelHashKey" FROM "varbinary"(32) NULL
-- Predicate: TypeDomain(sdLongTextString.CustomerComments) → varchar(200) ∪ {NULL}
    CREATE TYPE "sdLongTextString"."CustomerComments" FROM "varchar"(200) NULL
-- Predicate: TypeDomain(sdLongTextString.Note) → varchar(200) ∪ {NULL}
    CREATE TYPE "sdLongTextString"."Note" FROM "varchar"(200) NULL
-- Predicate: TypeDomain(sdMarketingTextString.CustomerSpendCapacity) → char(25) ∪ {NULL}
    CREATE TYPE "sdMarketingTextString"."CustomerSpendCapacity" FROM "char"(25) NULL
-- Predicate: TypeDomain(sdMarketingTextString.VehicleSalesThresholdCategory) → nvarchar(50) ∪ {NULL}
    CREATE TYPE "sdMarketingTextString"."VehicleSalesThresholdCategory" FROM "nvarchar"(50) NULL
-- Predicate: TypeDomain(sdOrdinalNumber.TransactionNumber) → int ∧ ∀x ∈ sdOrdinalNumber.TransactionNumber: x ≠ NULL
    CREATE TYPE "sdOrdinalNumber"."TransactionNumber" FROM "int" NOT NULL
-- Predicate: TypeDomain(sdPersonNameString.CustomerName) → varchar(60) ∪ {NULL}
    CREATE TYPE "sdPersonNameString"."CustomerName" FROM "varchar"(60) NULL
-- Predicate: TypeDomain(sdPersonNameString.FirstNameString) → varchar(30) ∧ ∀x ∈ sdPersonNameString.FirstNameString: x ≠ NULL
    CREATE TYPE "sdPersonNameString"."FirstNameString" FROM "varchar"(30) NOT NULL
-- Predicate: TypeDomain(sdPersonNameString.FullNameString) → varchar(65) ∧ ∀x ∈ sdPersonNameString.FullNameString: x ≠ NULL
    CREATE TYPE "sdPersonNameString"."FullNameString" FROM "varchar"(65) NOT NULL
-- Predicate: TypeDomain(sdPersonNameString.LastNameString) → varchar(35) ∧ ∀x ∈ sdPersonNameString.LastNameString: x ≠ NULL
    CREATE TYPE "sdPersonNameString"."LastNameString" FROM "varchar"(35) NOT NULL
-- Predicate: TypeDomain(sdSequenceNumber.LineItemNumber) → int ∧ ∀x ∈ sdSequenceNumber.LineItemNumber: x ≠ NULL
    CREATE TYPE "sdSequenceNumber"."LineItemNumber" FROM "int" NOT NULL
-- Predicate: TypeDomain(sdSequenceNumber.SurrogateKeyNumber) → int ∧ ∀x ∈ sdSequenceNumber.SurrogateKeyNumber: x ≠ NULL
    CREATE TYPE "sdSequenceNumber"."SurrogateKeyNumber" FROM "int" NOT NULL
-- Predicate: TypeDomain(sdSequenceNumber.UserAuthorizationID) → int ∧ ∀x ∈ sdSequenceNumber.UserAuthorizationID: x ≠ NULL
    CREATE TYPE "sdSequenceNumber"."UserAuthorizationID" FROM "int" NOT NULL
-- Predicate: TypeDomain(sdShortTextString.InvoiceNumber) → varchar(15) ∪ {NULL}
    CREATE TYPE "sdShortTextString"."InvoiceNumber" FROM "varchar"(15) NULL
-- Predicate: TypeDomain(sdShortTextString.StockCode) → nvarchar(50) ∪ {NULL}
    CREATE TYPE "sdShortTextString"."StockCode" FROM "nvarchar"(50) NULL
-- Predicate: TypeDomain(sdSysTime.AuditTriggerTimestamp) → datetime2(7) ∧ ∀x ∈ sdSysTime.AuditTriggerTimestamp: x ≠ NULL
    CREATE TYPE "sdSysTime"."AuditTriggerTimestamp" FROM "datetime2"(7) NOT NULL
-- Predicate: TypeDomain(sdSysTime.DateAdded) → datetime2(7) ∪ {NULL}
    CREATE TYPE "sdSysTime"."DateAdded" FROM "datetime2"(7) NULL
-- Predicate: TypeDomain(sdSysTime.DateOfLastUpdate) → datetime2(7) ∧ ∀x ∈ sdSysTime.DateOfLastUpdate: x ≠ NULL
    CREATE TYPE "sdSysTime"."DateOfLastUpdate" FROM "datetime2"(7) NOT NULL
-- Predicate: TypeDomain(sdSysTime.SysEndTime) → datetime2(7) ∧ ∀x ∈ sdSysTime.SysEndTime: x ≠ NULL
    CREATE TYPE "sdSysTime"."SysEndTime" FROM "datetime2"(7) NOT NULL
-- Predicate: TypeDomain(sdSysTime.SysStartTime) → datetime2(7) ∧ ∀x ∈ sdSysTime.SysStartTime: x ≠ NULL
    CREATE TYPE "sdSysTime"."SysStartTime" FROM "datetime2"(7) NOT NULL
-- Predicate: TypeDomain(sdVehicleDescriptorString.VehicleColor) → nvarchar(50) ∪ {NULL}
    CREATE TYPE "sdVehicleDescriptorString"."VehicleColor" FROM "nvarchar"(50) NULL
-- Predicate: TypeDomain(sdVehicleDescriptorString.VehicleIdentifier) → char(18) ∪ {NULL}
    CREATE TYPE "sdVehicleDescriptorString"."VehicleIdentifier" FROM "char"(18) NULL
-- Predicate: TypeDomain(sdVehicleDescriptorString.VehicleManufacturerMarketingType) → char(18) ∪ {NULL}
    CREATE TYPE "sdVehicleDescriptorString"."VehicleManufacturerMarketingType" FROM "char"(18) NULL
-- Predicate: TypeDomain(sdVehicleSalePayment.CostOfFee) → money ∧ ∀x ∈ sdVehicleSalePayment.CostOfFee: x ≠ NULL
    CREATE TYPE "sdVehicleSalePayment"."CostOfFee" FROM "money" NOT NULL
-- Predicate: TypeDomain(sdVehicleSalePayment.DiscountedPrice) → money ∧ ∀x ∈ sdVehicleSalePayment.DiscountedPrice: x ≠ NULL
    CREATE TYPE "sdVehicleSalePayment"."DiscountedPrice" FROM "money" NOT NULL
-- Predicate: TypeDomain(sdVehicleSalePayment.VehiclePrice) → int ∧ ∀x ∈ sdVehicleSalePayment.VehiclePrice: x ≠ NULL
    CREATE TYPE "sdVehicleSalePayment"."VehiclePrice" FROM "int" NOT NULL

-- Business context: Vehicle manufacturer data management
-- Predicate: Domain(ManufacturerVehicleMake) → ManufacturerVehicleMake ∈ Inventory
CREATE TABLE "Inventory"."ManufacturerVehicleMake"(
    -- Column predicate: ManufacturerVehicleMakeID: ManufacturerVehicleMake → SurrogateKeyNumber
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: ManufacturerVehicleMakeID(x) ≠ NULL
    "ManufacturerVehicleMakeID" "sdSequenceNumber"."SurrogateKeyNumber" NOT NULL,
    
    -- Column predicate: CountryId: ManufacturerVehicleMake → SurrogateKeyNumber ∪ {NULL}
    "CountryId" "sdSequenceNumber"."SurrogateKeyNumber" NULL,
    
    -- Column predicate: ManufacturerVehicleMakeName: ManufacturerVehicleMake → VehicleIdentifier ∪ {NULL}
    "ManufacturerVehicleMakeName" "sdVehicleDescriptorString"."VehicleIdentifier" NULL,
    
    -- Column predicate: ManufacturerVehicleMarketingCategoryID: ManufacturerVehicleMake → SurrogateKeyNumber
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: ManufacturerVehicleMarketingCategoryID(x) ≠ NULL
    "ManufacturerVehicleMarketingCategoryID" "sdSequenceNumber"."SurrogateKeyNumber" NOT NULL,
    
    -- Column predicate: Note: ManufacturerVehicleMake → Note
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: Note(x) ≠ NULL
    "Note" "sdLongTextString"."Note" NOT NULL,
    
    -- Column predicate: TransactionNumber: ManufacturerVehicleMake → TransactionNumber
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: TransactionNumber(x) ≠ NULL
    "TransactionNumber" "sdOrdinalNumber"."TransactionNumber" NOT NULL,
    
    -- Column predicate: UserAuthorizationID: ManufacturerVehicleMake → UserAuthorizationID
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: UserAuthorizationID(x) ≠ NULL
    "UserAuthorizationID" "sdSequenceNumber"."UserAuthorizationID" NOT NULL,
    
    -- Column predicate: SysStartTime: ManufacturerVehicleMake → SysStartTime
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: SysStartTime(x) ≠ NULL
    "SysStartTime" "sdSysTime"."SysStartTime" NOT NULL,
    
    -- Column predicate: SysEndTime: ManufacturerVehicleMake → SysEndTime
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: SysEndTime(x) ≠ NULL
    "SysEndTime" "sdSysTime"."SysEndTime" NOT NULL,
    
    -- Column predicate: RowLevelHashKey: ManufacturerVehicleMake → RowLevelHashKey ∪ {NULL}
    "RowLevelHashKey" "sdHashKey"."RowLevelHashKey" NULL,
    
    -- Column predicate: PriorRowLevelHashKey: ManufacturerVehicleMake → RowLevelHashKey ∪ {NULL}
    "PriorRowLevelHashKey" "sdHashKey"."RowLevelHashKey" NULL,
    
    -- Column predicate: FireAuditTrigger: ManufacturerVehicleMake → FlagYesNoString
    -- Non-null predicate: ∀x ∈ ManufacturerVehicleMake: FireAuditTrigger(x) ≠ NULL
    "FireAuditTrigger" "sdFlagString"."FlagYesNoString" NOT NULL,
    
    -- Primary Key predicate: ∀x,y ∈ ManufacturerVehicleMake: (x.ManufacturerVehicleMakeID = y.ManufacturerVehicleMakeID) → (x = y)
    -- Existence predicate: ∀x ∈ ManufacturerVehicleMake: ∃!id (id = x.ManufacturerVehicleMakeID)
 CONSTRAINT "PK_Make" PRIMARY KEY CLUSTERED 
(
    "ManufacturerVehicleMakeID" ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON "PRIMARY"
) ON "PRIMARY"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.ManufacturerVehicleMakeID = NULL → x.ManufacturerVehicleMakeID := NEXT VALUE FOR PKSequence.InventoryManufacturerVehicleMakeSeqId
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_ManufacturerVehicleMakeID"  DEFAULT (NEXT VALUE FOR "PKSequence"."InventoryManufacturerVehicleMakeSeqId") FOR "ManufacturerVehicleMakeID"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.CountryId = NULL → x.CountryId := 'United Kingdom'
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_CountryId"  DEFAULT ('United Kingdom') FOR "CountryId"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.Note = NULL → x.Note := 'Row was inserted'
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_Note"  DEFAULT ('Row was inserted') FOR "Note"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.TransactionNumber = NULL → x.TransactionNumber := 1
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_TransactionNumber"  DEFAULT ((1)) FOR "TransactionNumber"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.UserAuthorizationID = NULL → x.UserAuthorizationID := 1
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_UserAuthorizationID"  DEFAULT ((1)) FOR "UserAuthorizationID"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.SysStartTime = NULL → x.SysStartTime := sysdatetime()
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_SysStartTime"  DEFAULT (sysdatetime()) FOR "SysStartTime"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.SysEndTime = NULL → x.SysEndTime := '99991231 23:59:59.9999999'
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_SysEndTime"  DEFAULT ('99991231 23:59:59.9999999') FOR "SysEndTime"

-- Default value predicate: ∀x ∈ ManufacturerVehicleMake: x.FireAuditTrigger = NULL → x.FireAuditTrigger := 'N'
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" ADD  CONSTRAINT "DF_Inventory_ManufacturerVehicleMake_FireAuditTrigger"  DEFAULT ('N') FOR "FireAuditTrigger"

-- Check constraint predicate: ∀x ∈ ManufacturerVehicleMake: FireAuditTrigger(x) = 'Y' ∨ FireAuditTrigger(x) = 'N'
    ALTER TABLE "Inventory"."ManufacturerVehicleMake"  WITH CHECK ADD  CONSTRAINT "CK_Inventory_ManufacturerVehicleMake_FireAuditTrigger" CHECK  (("FireAuditTrigger"='Y' OR "FireAuditTrigger"='N'))
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" CHECK CONSTRAINT "CK_Inventory_ManufacturerVehicleMake_FireAuditTrigger"

-- Check constraint predicate: ∀x ∈ ManufacturerVehicleMake: SysStartTime(x) < SysEndTime(x)
    ALTER TABLE "Inventory"."ManufacturerVehicleMake"  WITH CHECK ADD  CONSTRAINT "XCK_Inventory_ManufacturerVehicleMake_SysEndTime" CHECK  (("SysStartTime"<"SysEndTime"))
    ALTER TABLE "Inventory"."ManufacturerVehicleMake" CHECK CONSTRAINT "XCK_Inventory_ManufacturerVehicleMake_SysEndTime"

-- Business context: Audit history for vehicle manufacturer data
-- Predicate: Domain(InventoryManufacturerVehicleMakeHistory) → InventoryManufacturerVehicleMakeHistory ∈ Audit
CREATE TABLE "Audit"."InventoryManufacturerVehicleMakeHistory"(
    -- Column predicate: InventoryManufacturerVehicleMakeHistoryId: InventoryManufacturerVehicleMakeHistory → SurrogateKeyNumber
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: InventoryManufacturerVehicleMakeHistoryId(x) ≠ NULL
    "InventoryManufacturerVehicleMakeHistoryId" "sdSequenceNumber"."SurrogateKeyNumber" NOT NULL,
    
    -- Column predicate: AuditDateTimeStamp: InventoryManufacturerVehicleMakeHistory → AuditTriggerTimestamp
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: AuditDateTimeStamp(x) ≠ NULL
    "AuditDateTimeStamp" "sdSysTime"."AuditTriggerTimestamp" NOT NULL,
    
    -- Column predicate: DBAction: InventoryManufacturerVehicleMakeHistory → DbAction
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: DBAction(x) ≠ NULL
    "DBAction" "sdAudit"."DbAction" NOT NULL,
    
    -- Column predicate: ManufacturerVehicleMakeID: InventoryManufacturerVehicleMakeHistory → SurrogateKeyNumber
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: ManufacturerVehicleMakeID(x) ≠ NULL
    "ManufacturerVehicleMakeID" "sdSequenceNumber"."SurrogateKeyNumber" NOT NULL,
    
    -- Column predicate: ManufacturerVehicleMakeName: InventoryManufacturerVehicleMakeHistory → VehicleIdentifier ∪ {NULL}
    "ManufacturerVehicleMakeName" "sdVehicleDescriptorString"."VehicleIdentifier" NULL,
    
    -- Column predicate: CountryID: InventoryManufacturerVehicleMakeHistory → SurrogateKeyNumber ∪ {NULL}
    "CountryID" "sdSequenceNumber"."SurrogateKeyNumber" NULL,
    
    -- Column predicate: ManufacturerVehicleMarketingCategoryID: InventoryManufacturerVehicleMakeHistory → SurrogateKeyNumber
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: ManufacturerVehicleMarketingCategoryID(x) ≠ NULL
    "ManufacturerVehicleMarketingCategoryID" "sdSequenceNumber"."SurrogateKeyNumber" NOT NULL,
    
    -- Column predicate: Note: InventoryManufacturerVehicleMakeHistory → Note
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: Note(x) ≠ NULL
    "Note" "sdLongTextString"."Note" NOT NULL,
    
    -- Column predicate: TransactionNumber: InventoryManufacturerVehicleMakeHistory → TransactionNumber
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: TransactionNumber(x) ≠ NULL
    "TransactionNumber" "sdOrdinalNumber"."TransactionNumber" NOT NULL,
    
    -- Column predicate: UserAuthorizationID: InventoryManufacturerVehicleMakeHistory → UserAuthorizationID
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: UserAuthorizationID(x) ≠ NULL
    "UserAuthorizationID" "sdSequenceNumber"."UserAuthorizationID" NOT NULL,
    
    -- Column predicate: SysStartTime: InventoryManufacturerVehicleMakeHistory → SysStartTime
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: SysStartTime(x) ≠ NULL
    "SysStartTime" "sdSysTime"."SysStartTime" NOT NULL,
    
    -- Column predicate: SysEndTime: InventoryManufacturerVehicleMakeHistory → SysEndTime
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: SysEndTime(x) ≠ NULL
    "SysEndTime" "sdSysTime"."SysEndTime" NOT NULL,
    
    -- Column predicate: RowLevelHashKey: InventoryManufacturerVehicleMakeHistory → RowLevelHashKey ∪ {NULL}
    "RowLevelHashKey" "sdHashKey"."RowLevelHashKey" NULL,
    
    -- Column predicate: PriorRowLevelHashKey: InventoryManufacturerVehicleMakeHistory → RowLevelHashKey ∪ {NULL}
    "PriorRowLevelHashKey" "sdHashKey"."RowLevelHashKey" NULL,
    
    -- Column predicate: FireAuditTrigger: InventoryManufacturerVehicleMakeHistory → FlagYesNoString
    -- Non-null predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: FireAuditTrigger(x) ≠ NULL
    "FireAuditTrigger" "sdFlagString"."FlagYesNoString" NOT NULL,
    
    -- Primary Key predicate: ∀x,y ∈ InventoryManufacturerVehicleMakeHistory: (x.InventoryManufacturerVehicleMakeHistoryId = y.InventoryManufacturerVehicleMakeHistoryId) → (x = y)
    -- Existence predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: ∃!id (id = x.InventoryManufacturerVehicleMakeHistoryId)
 CONSTRAINT "PK_Make" PRIMARY KEY NONCLUSTERED 
(
    "InventoryManufacturerVehicleMakeHistoryId" ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON "PRIMARY",
    
    -- Unique constraint predicate: ∀x,y ∈ InventoryManufacturerVehicleMakeHistory: (x ≠ y) → (x.ManufacturerVehicleMakeID ≠ y.ManufacturerVehicleMakeID ∨ x.TransactionNumber ≠ y.TransactionNumber)
 CONSTRAINT "XAK1_InventoryManufacturerVehicleMakeHistory" UNIQUE CLUSTERED 
(
    "ManufacturerVehicleMakeID" ASC,
    "TransactionNumber" DESC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON "PRIMARY"
) ON "PRIMARY"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.InventoryManufacturerVehicleMakeHistoryId = NULL → x.InventoryManufacturerVehicleMakeHistoryId := NEXT VALUE FOR PKSequence.AuditInventoryManufacturerVehicleMakeHistorySeqId
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_InventoryManufacturerVehicleMakeHistoryId"  DEFAULT (NEXT VALUE FOR "PKSequence"."AuditInventoryManufacturerVehicleMakeHistorySeqId") FOR "InventoryManufacturerVehicleMakeHistoryId"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.AuditDateTimeStamp = NULL → x.AuditDateTimeStamp := sysdatetime()
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_AuditDateTimeStamp"  DEFAULT (sysdatetime()) FOR "AuditDateTimeStamp"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.DBAction = NULL → x.DBAction := 'U'
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_DBAction"  DEFAULT ('U') FOR "DBAction"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.Note = NULL → x.Note := 'No Comment Provided'
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_Note"  DEFAULT ('No Comment Provided') FOR "Note"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.TransactionNumber = NULL → x.TransactionNumber := 1
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_TransactionNumber"  DEFAULT ((1)) FOR "TransactionNumber"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.UserAuthorizationID = NULL → x.UserAuthorizationID := 1
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_UserAuthorizationID"  DEFAULT ((1)) FOR "UserAuthorizationID"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.SysStartTime = NULL → x.SysStartTime := sysdatetime()
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_SysStartTime"  DEFAULT (sysdatetime()) FOR "SysStartTime"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.SysEndTime = NULL → x.SysEndTime := '99991231 23:59:59.9999999'
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_SysEndTime"  DEFAULT ('99991231 23:59:59.9999999') FOR "SysEndTime"

-- Default value predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: x.FireAuditTrigger = NULL → x.FireAuditTrigger := 'N'
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" ADD  CONSTRAINT "DF_Audit_InventoryManufacturerVehicleMakeHistory_FireAuditTrigger"  DEFAULT ('N') FOR "FireAuditTrigger"

-- Check constraint predicate: ∀x ∈ InventoryManufacturerVehicleMakeHistory: SysStartTime(x) < SysEndTime(x)
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory"  WITH CHECK ADD  CONSTRAINT "XCK_Audit_InventoryManufacturerVehicleMakeHistory_SysEndTime" CHECK  (("SysStartTime"<"SysEndTime"))
ALTER TABLE "Audit"."InventoryManufacturerVehicleMakeHistory" CHECK CONSTRAINT "XCK_Audit_InventoryManufacturerVehicleMakeHistory_SysEndTime"
