# Houseing-Data-for-Cleaning
How to clean Dataset Useing  SQL
/****** For Analysis First Need To clean Data for Better Analyzing ******/

> SElECT * FROM HousingData

/*
Cleaning HousingData in SQL Queries          */

SElECT * 
FROM HousingData


/*  Standardize SaleDate Format To Date   */

Select saleDate, CONVERT(Date,SaleDate)
From HousingData


Update HousingData
SET SaleDate = CONVERT(Date,SaleDate)

-- But doesn't Update properly So use Alter First

ALTER TABLE HousingData
Add SaleDateConverted Date

Update HousingData
SET SaleDateConverted = CONVERT(Date,SaleDate)

SElECT * 
FROM HousingData  
-- All things Done Properly


/* Populate Property Address data          */

Select *
From HousingData
--Where PropertyAddress is null
order by ParcelID



Select a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress,b.PropertyAddress)
From HousingData a
JOIN HousingData b
	on a.ParcelID = b.ParcelID
	AND a.[UniqueID ] <> b.[UniqueID ]
Where a.PropertyAddress is null


Update a
SET PropertyAddress = ISNULL(a.PropertyAddress,b.PropertyAddress)
From HousingData a
JOIN HousingData b
	on a.ParcelID = b.ParcelID
	AND a.[UniqueID ] <> b.[UniqueID ]
Where a.PropertyAddress is null





/* Breaking out Address into Individual Columns (Address, City, State) */


Select PropertyAddress
From HousingData
--Where PropertyAddress is null
--order by ParcelID

SELECT
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1 ) as Address
, SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1 , LEN(PropertyAddress)) as Address

From HousingData


ALTER TABLE HousingData
Add PropertySplitAddress Nvarchar(255)

Update HousingData
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1 )


ALTER TABLE HousingData
Add PropertySplitCity Nvarchar(255)

Update HousingData
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1 , LEN(PropertyAddress))




Select *
From HousingData





Select OwnerAddress
From HousingData


Select
PARSENAME(REPLACE(OwnerAddress, ',', '.') , 3)
,PARSENAME(REPLACE(OwnerAddress, ',', '.') , 2)
,PARSENAME(REPLACE(OwnerAddress, ',', '.') , 1)
From HousingData



ALTER TABLE HousingData
Add OwnerSplitAddress Nvarchar(255)

Update HousingData
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.') , 3)


ALTER TABLE HousingData
Add OwnerSplitCity Nvarchar(255)

Update HousingData
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.') , 2)



ALTER TABLE HousingData
Add OwnerSplitState Nvarchar(255)

Update HousingData
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.') , 1)



Select *
From HousingData





/* Change Y and N to Yes and No in "Sold as Vacant" field      */


Select Distinct(SoldAsVacant), Count(SoldAsVacant)
From HousingData
Group by SoldAsVacant
order by 2




Select SoldAsVacant
, CASE When SoldAsVacant = 'Y' THEN 'Yes'
	   When SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END
From HousingData


Update HousingData
SET SoldAsVacant = CASE When SoldAsVacant = 'Y' THEN 'Yes'
	   When SoldAsVacant = 'N' THEN 'No'
	   ELSE SoldAsVacant
	   END







/*  Remove Duplicates             */
-- First find  pop up duplicate useing row_no 
-- Then Row_no >1 ko extract then remove

WITH RowNumCTE AS(
Select *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num

From HousingData
--order by ParcelID
)
Select *
From RowNumCTE
Where row_num > 1
Order by PropertyAddress



-- Here Useing Delete Keywords
WITH RowNumCTE AS(
Select *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num

From HousingData
)
DELETE
From RowNumCTE
Where row_num > 1



Select *
From HousingData




/* Useing Dataset  Delete Unused Columns For Preparation               */



Select *
From HousingData


ALTER TABLE HousingData
DROP COLUMN PropertyAddress, SaleDate,OwnerAddress, TaxDistrict


Select *
From HousingData 
