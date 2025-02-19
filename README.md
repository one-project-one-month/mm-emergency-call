# MM-Emergency-Call
ဒီ project က အရေးပေါ်အခြေနေတစ်ခုခုကြုံလာရင် လွယ်ကူစွာ လိုအပ်တဲ့ service တွေရှာနိုင်ဖို့အတွက် ဖန်တီးထားတဲ့ Idea တစ်ခုဖြစ်ပါတယ်။

Frontend
- [React.js with TypeScript](https://github.com/one-project-one-month/mm-emergency-call-react)

Backend
- [C#](https://github.com/one-project-one-month/mm-emergency-call-csharp)

ကျွန်တော်တို့က ဒီ Project အတွက် လိုအပ်တဲ့ Tableလေးတွေ တည်ဆောက်ထားတယ်။ ကိုယ်လိုအပ်သလို ထပ်ကွန့်ပြီး ချဲ့လို့ရပါတယ်။ ဒီ Project က တစ်လအတွင်း လုပ်ရတာဖြစ်တဲ့အတွက်
အမြန် အသုံးပြုလို့ရမယ့် Table တွေပဲ အရင် ထုတ်ထားပါတယ်။

1. [Users](#users)
2. [Emergency Services](#emergency services)
3. [Emergency Requests](#emergency requests)
4. [State Regions](#state regions)
5. [Townships](#Townships)

## Users
User Table ကို ကျွန်တော်တို့ project ကိုသုံးတဲ့သူတွေ ကျွန်တောိတို့ကိုserviceပေးတဲ့သူတွေရဲ့informationကိုသိမ်းဖို့တည်ဆောတ်ထားပါတယ်။ 
```sql
CREATE TABLE [dbo].[Users](
	[UserId] [int] IDENTITY(1,1) NOT NULL,
	[Name] [nvarchar](200) NOT NULL,
	[Email] [nvarchar](50) NOT NULL,
	[Password] [nvarchar](100) NOT NULL,
	[PhoneNumber] [nvarchar](20) NOT NULL,
	[Address] [nvarchar](300) NOT NULL,
	[EmergencyType] [nvarchar](50) NULL,
	[EmergencyDetails] [nvarchar](max) NULL,
	[TownshipCode] [nvarchar](100) NOT NULL,
	[UserStatus] [nvarchar](100) NULL,
	[Role] [nvarchar](100) NOT NULL,
	[IsVerified] [char](1) NULL,
	[OTP] [nvarchar](50) NULL)
```

## [Emergency Services]
Emergency Services Table ကို emergency case အမျိုးမျိုးအတွက်လိုအပ်သော emergency service များကို သိမ်းဖို့အတွတ်တည်ဆောတ်ထားပါတယ်။
```sql
CREATE TABLE [dbo].[EmergencyServices](
	[ServiceId] [int] IDENTITY(1,1) NOT NULL,
	[UserId] [int] NOT NULL,
	[ServiceGroup] [nvarchar](50) NOT NULL,
	[ServiceType] [nvarchar](50) NOT NULL,
	[ServiceName] [nvarchar](100) NOT NULL,
	[PhoneNumber] [nvarchar](15) NOT NULL,
	[Location] [nvarchar](max) NULL,
	[Availability] [char](1) NULL,
	[TownshipCode] [nvarchar](200) NULL,
	[ServiceStatus] [varchar](10) NULL,
	[Lng] [decimal](18, 7) NULL,
	[ltd] [decimal](18, 7) NULL)
```

## [Emergency Requests]
Emergency Request Tableကို User တွေက emergency service ကိုဆက်သွယ်တဲ့ request time ရယ် respond time တွေရယ် အကြိမ်ရေတွေကိုသိမ်းဖို့တည်ဆောတ်ထားပါတယ်။
```sql
CREATE TABLE [dbo].[EmergencyRequests](
	[RequestId] [int] IDENTITY(1,1) NOT NULL,
	[UserId] [int] NOT NULL,
	[ServiceId] [int] NOT NULL,
	[ProviderId] [int] NULL,
	[RequestTime] [datetime] NOT NULL,
	[Status] [nvarchar](50) NOT NULL,
	[ResponseTime] [datetime] NULL,
	[Notes] [text] NULL,
	[TownshipCode] [nvarchar](100) NULL)
```

## State Regions
State Regions Table ကို emergency service တွေရှိတဲ့ States ကိုသိမ်းဖို့တည်ဆောတ်ထားပါတယ်။
```sql
CREATE TABLE [dbo].[StateRegions](
	[StateRegionId] [int] IDENTITY(1,1) NOT NULL,
	[StateRegionCode] [nvarchar](50) NOT NULL,
	[StateRegionNameEn] [nvarchar](200) NOT NULL,
	[StateRegionNameMm] [nvarchar](200) NULL)
```

## Townships
Townships Table ကို emergency service ရှိတဲ့ Townships ကိုသိမ်းဖို့တည်ဆောတ်ထားပါတယ်။
```sql
CREATE TABLE [dbo].[Townships](
	[TownshipId] [int] IDENTITY(1,1) NOT NULL,
	[TownshipCode] [nvarchar](50) NOT NULL,
	[TownshipNameEn] [nvarchar](200) NOT NULL,
	[TownshipNameMm] [nvarchar](200) NOT NULL,
	[StateRegionCode] [nvarchar](50) NOT NULL)
```

## Store Procedure for Dashboard
```sql
-----------------------------------
-- Test Script
-----------------------------------
/*
DBCC DROPCLEANBUFFERS
GO 

DECLARE @UserId			nvarchar(50)	SET @UserId='HEIN'

EXEC sp_Dashboard_DailyWeeklyMonthlyPerRequest	@UserId

select * from EmergencyRequests
*/

------------------------------------------------------------------------------------------------------------------------------------------------------------
-- Change History
------------------------------------------------------------------------------------------------------------------------------------------------------------
-- 15/De/2024 HEIN	- create the sp for Dashboard
------------------------------------------------------------------------------------------------------------------------------------------------------------

CREATE PROCEDURE [dbo].[sp_Dashboard_DailyWeeklyMonthlyPerRequest]    
	( 
	@UserId					nvarchar(50)
	) 
AS
BEGIN
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	-- Create Temp Table to Return Data 
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	CREATE TABLE #RequestSummary
	(	
		DailyRequest INT,
		WeeklyRequest INT,
		MonthlyRequest INT
	);
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	-- Retrieving Daily Request
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	DECLARE @DailyRequest INT  SET @DailyRequest = 0
	SELECT @DailyRequest = COUNT(*) FROM EmergencyRequests (NOLOCK) WHERE CONVERT(DATE,RequestTime) = CONVERT(DATE,GETDATE())

	------------------------------------------------------------------------------------------------------------------------------------------------------------
	-- Retrieving Weekly Request
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	DECLARE @WeeklyRequest INT  SET @WeeklyRequest = 0
	SELECT @WeeklyRequest = COUNT(*) FROM EmergencyRequests (NOLOCK) WHERE CONVERT(DATE,RequestTime) BETWEEN CONVERT(DATE,GETDATE()-7) AND CONVERT(DATE,GETDATE())

	------------------------------------------------------------------------------------------------------------------------------------------------------------
	-- Retrieving Monthly Request
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	DECLARE @MonthlyRequest INT  SET @MonthlyRequest = 0
	SELECT @MonthlyRequest= COUNT(*) FROM EmergencyRequests WHERE CONVERT(DATE,RequestTime) BETWEEN CONVERT(DATE,DATEADD(month, -1, GETDATE())) AND CONVERT(DATE,GETDATE())

	INSERT INTO #RequestSummary(DailyRequest, WeeklyRequest, MonthlyRequest) VALUES 
		(@DailyRequest, @WeeklyRequest, @MonthlyRequest )

	SELECT * FROM #RequestSummary
	DROP TABLE #RequestSummary
END

---------------------------------------------------------------------------------------------
--END Of sp_Dashboard_DailyWeeklyMonthlyPerRequest 
---------------------------------------------------------------------------------------------

GO
/****** Object:  StoredProcedure [dbo].[sp_Dashboard_Process]    Script Date: 1/26/2025 11:00:07 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

-----------------------------------
-- Test Script
-----------------------------------
/*
DBCC DROPCLEANBUFFERS
GO 

DECLARE @UserId			nvarchar(50)	SET @UserId='HEIN'

EXEC sp_Dashboard_Process	@UserId

*/

------------------------------------------------------------------------------------------------------------------------------------------------------------
-- Change History
------------------------------------------------------------------------------------------------------------------------------------------------------------
-- 15/De/2024 HEIN	- create the sp for Dashboard
------------------------------------------------------------------------------------------------------------------------------------------------------------

CREATE PROCEDURE [dbo].[sp_Dashboard_Process]    
	( 
	@UserId					nvarchar(50)
	) 
AS
BEGIN

	EXEC dbo.sp_Dashboard_DailyWeeklyMonthlyPerRequest		@UserId
	
	EXEC dbo.sp_Dashboard_TopTenServicePerTownship			@UserId

  EXEC dbo.sp_Dashboard_ServiceProviderActivity	
  
  EXEC dbo.sp_Dashboard_TopTenRequestPerUser

  EXEC dbo.sp_Dashboard_TopTenServicePerTownship  
	
END


GO
/****** Object:  StoredProcedure [dbo].[sp_Dashboard_ServiceProviderActivity]    Script Date: 1/26/2025 11:00:07 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE [dbo].[sp_Dashboard_ServiceProviderActivity]

AS

BEGIN
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	-- Create Temp Table to Return Data 
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	CREATE TABLE #ServiceProviderActivity
	(	
		ServiceId		INT,
		ServiceName		NVARCHAR(100),
		ServiceType		NVARCHAR(50),
		PhoneNumber		NVARCHAR(15),
		Location		NVARCHAR(MAX),
		Count			INT
	);
	
	INSERT INTO #ServiceProviderActivity
	(ServiceId, ServiceName, ServiceType, PhoneNumber, Location, Count ) 

	SELECT ER.ServiceId
			,ES.ServiceName
			,ES.ServiceType
			,ES.PhoneNumber
			,ES.Location
			,Count(ER.ServiceId) AS ServiceCount
	FROM EmergencyRequests AS ER
	INNER JOIN EmergencyServices AS ES ON ES.ServiceId = ER.ServiceId
	WHERE Status = 'Completed'
	GROUP BY ER.ServiceId
		,ES.ServiceName
	    ,ES.[ServiceType]
	    ,ES.[PhoneNumber]
	    ,ES.[Location]
	ORDER BY ServiceCount DESC

	SELECT * FROM #ServiceProviderActivity
	DROP TABLE #ServiceProviderActivity
END

---------------------------------------------------------------------------------------------
--END Of [sp_Dashboard_ServiceProviderActivity]
---------------------------------------------------------------------------------------------

GO
/****** Object:  StoredProcedure [dbo].[sp_Dashboard_TopTenRequestPerUser]    Script Date: 1/26/2025 11:00:07 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE [dbo].[sp_Dashboard_TopTenRequestPerUser]

AS

BEGIN
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	-- Create Temp Table to Return Data 
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	CREATE TABLE #TopTenRequestPerUser
	(	
		UserId			INT,
		UserName		NVARCHAR(200),
		UserEmail		NVARCHAR(50),
		UserPhoneNumber	NVARCHAR(20),
		UserAddress		NVARCHAR(300),
		Count			INT
	);
	
	INSERT INTO #TopTenRequestPerUser
	(UserId, UserName, UserEmail, UserPhoneNumber, UserAddress, Count ) 

	SELECT TOP 10 
	    U.UserId, 
	    U.Name, 
	    U.Email,
		U.PhoneNumber,
		U.Address,
	    COUNT(U.UserId) AS UserCount
	FROM EmergencyRequests AS ER
	INNER JOIN Users AS U ON U.UserId = ER.UserId
	INNER JOIN EmergencyServices AS ES ON ES.ServiceId = ER.ServiceId
	GROUP BY 
	    U.UserId, 
	    U.Name, 
	    U.Email,
		U.PhoneNumber,
		U.Address
	ORDER BY UserCount DESC

	SELECT * FROM #TopTenRequestPerUser
	DROP TABLE #TopTenRequestPerUser
END

---------------------------------------------------------------------------------------------
--END Of [sp_Dashboard_TopTenRequestPerUser]
---------------------------------------------------------------------------------------------

GO
/****** Object:  StoredProcedure [dbo].[sp_Dashboard_TopTenServicePerTownship]    Script Date: 1/26/2025 11:00:07 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


-----------------------------------
-- Test Script
-----------------------------------
/*
DBCC DROPCLEANBUFFERS
GO 

DECLARE @UserId			nvarchar(50)	SET @UserId='HEIN'

EXEC sp_Dashboard_TopTenServicePerTownship	@UserId

*/

------------------------------------------------------------------------------------------------------------------------------------------------------------
-- Change History
------------------------------------------------------------------------------------------------------------------------------------------------------------
-- 17/De/2024 Hnin Wutt Yi	- create the sp for Dashboard
------------------------------------------------------------------------------------------------------------------------------------------------------------
-- 18/De/2024 Hnin Wutt Yi	- update the sp for Dashboard
------------------------------------------------------------------------------------------------------------------------------------------------------------

CREATE PROCEDURE [dbo].[sp_Dashboard_TopTenServicePerTownship]    
	( 
	@UserId					nvarchar(50)
	) 
AS
BEGIN
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	-- Create Temp Table to Return Data 
	------------------------------------------------------------------------------------------------------------------------------------------------------------
	CREATE TABLE #TopTenServicePerTownship
	(	
		TownshipCode	 NVARCHAR(100),
		TownshipNameEn	 NVARCHAR(300),
		TownshipNameMM	 NVARCHAR(300),
		Count				INT
	);
	
	INSERT INTO #TopTenServicePerTownship
	(TownshipCode, TownshipNameEn, TownshipNameMM, Count ) 
	SELECT TOP 10 T.TownshipCode,T.TownshipNameEn, T.TownshipNameMm,COUNT(T.TownshipCode) from EmergencyServices AS ES
    INNER JOIN Townships AS T ON T.TOWNSHIPCODE = ES.TownshipCode GROUP BY T.TownshipCode,T.TownshipNameEn, T.TownshipNameMm ORDER BY COUNT(T.TownshipCode) DESC


	SELECT * FROM #TopTenServicePerTownship
	DROP TABLE #TopTenServicePerTownship
END

---------------------------------------------------------------------------------------------
--END Of sp_Dashboard_TopTenServicePerTownship 
---------------------------------------------------------------------------------------------

GO

```
