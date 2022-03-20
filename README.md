# Housing Data Cleaning
In this project I take raw housing dataset and transform it in SQL to make it more useful and more usable for analysis.

## Tool used
- Bigquery SQL

## Data Sourcing and processing
- Get raw [housing data](https://drive.google.com/drive/u/3/folders/149sNQs1ZrmIqImHKL5-ZvN1qbXs5n7SA)
- Import data into bigquery sql
- Populate property address data
```sql
SELECT
    *
FROM
    `project-for-portfolio.data_cleaning_project.houses`
WHERE
    PropertyAddress is null
ORDER BY
    ParcelID;


update `project-for-portfolio.data_cleaning_project.houses` a
set a.PropertyAddress = b.PropertyAddress
from (
    select ParcelID, min(PropertyAddress) as PropertyAddress from `project-for-portfolio.data_cleaning_project.houses` where PropertyAddress is not null group by ParcelID
) as b
where a.ParcelID = b.ParcelID and a.PropertyAddress is null;
```
- Breaking out Property address into individual columns (Address, City)
```sql
select
    PropertyAddress
from
    `project-for-portfolio.data_cleaning_project.houses`;

select
    split(PropertyAddress, ',') [safe_offset(0)] as Address,
    split(PropertyAddress, ',') [safe_offset(1)] as City
from
    `project-for-portfolio.data_cleaning_project.houses`;



Alter Table
    `project-for-portfolio.data_cleaning_project.houses`
Add Column
    `PropertySplitAddress` string (250);

update
    `project-for-portfolio.data_cleaning_project.houses`
set
    PropertySplitAddress = split(PropertyAddress, ',') [safe_offset(0)]
where
    true;
    


Alter Table
    `project-for-portfolio.data_cleaning_project.houses`
Add Column
    `PropertySplitCity` string (250);

update
    `project-for-portfolio.data_cleaning_project.houses`
set
    PropertySplitCity = split(PropertyAddress, ',') [safe_offset(1)]
where
    true;
```
- Breaking out Owners Address into individual column (Address, City, State)
```sql
select
    split(OwnerAddress, ',') [safe_offset(0)] as Address,
    split(OwnerAddress, ',') [safe_offset(1)] as City,
    split(OwnerAddress, ',') [safe_offset(2)] as State
from
    `project-for-portfolio.data_cleaning_project.houses`;



Alter Table
    `project-for-portfolio.data_cleaning_project.houses`
Add Column
    `OwnerSplitAddress` string (250);

update
    `project-for-portfolio.data_cleaning_project.houses`
set
    OwnerSplitAddress = split(OwnerAddress, ',') [safe_offset(0)]
where
    true;
    


Alter Table
    `project-for-portfolio.data_cleaning_project.houses`
Add Column
    `OwnerSplitCity` string (250);

update
    `project-for-portfolio.data_cleaning_project.houses`
set
    OwnerSplitCity = split(OwnerAddress, ',') [safe_offset(1)]
where
    true;



Alter Table
    `project-for-portfolio.data_cleaning_project.houses`
Add Column
    `OwnerSplitState` string (250);

update
    `project-for-portfolio.data_cleaning_project.houses`
set
    OwnerSplitState = split(OwnerAddress, ',') [safe_offset(2)]
where
    true;
```
- Change Y and N to Yes and No in 'Sold as vaccant' field
```sql
select
    distinct(SoldAsVacant), count(SoldAsVacant)
from
    `project-for-portfolio.data_cleaning_project.houses`
group by
    SoldAsVacant
order by 2;


select
    SoldAsVacant,
    case when SoldAsVacant = 'Y' then 'Yes'
         when SoldAsVacant = 'N' then 'No'
         else SoldAsVacant
         end
from
    `project-for-portfolio.data_cleaning_project.houses`;


update
    `project-for-portfolio.data_cleaning_project.houses`
set SoldAsVacant = case when SoldAsVacant = 'Y' then 'Yes'
                        when SoldAsVacant = 'N' then 'No'
                        else SoldAsVacant
                        end
where true;


update
    `project-for-portfolio.data_cleaning_project.houses`
set SoldAsVacant = case when SoldAsVacant = '0  COUCHVILLE PIKE, HERMITAGE, TN' then 'Yes'
                        when SoldAsVacant = '142  SCENIC VIEW RD, OLD HICKORY, TN' then 'No'
                        when SoldAsVacant = '144  SCENIC VIEW RD, OLD HICKORY, TN' then 'No'
                        else SoldAsVacant
                        end
where true;



select
    SoldAsVacant,
    case when SoldAsVacant = '0  COUCHVILLE PIKE, HERMITAGE, TN' then 'Yes'
         when SoldAsVacant = '142  SCENIC VIEW RD, OLD HICKORY, TN' then 'No'
         when SoldAsVacant = '144  SCENIC VIEW RD, OLD HICKORY, TN' then 'No'
         else SoldAsVacant
         end
from
    `project-for-portfolio.data_cleaning_project.houses`;
```
- Removed duplicates
```sql
with RowNumCTE as(
    select *,
           row_number() over(
               partition by ParcelID,
                            PropertyAddress,
                            SalePrice,
                            SaleDate,
                            LegalReference
                            order by
                                UniqueID_
           ) row_num
from
    `project-for-portfolio.data_cleaning_project.houses`
)
select *
from
    RowNumCTE
where
    row_num > 1
order by
    PropertyAddress;



DELETE FROM `project-for-portfolio.data_cleaning_project.houses` d
WHERE EXISTS (
        SELECT *
        FROM `project-for-portfolio.data_cleaning_project.houses` x
        WHERE x.ParcelID = d.ParcelID
        AND x.PropertyAddress = d.PropertyAddress
        AND x.SalePrice = d.SalePrice
        AND x.SaleDate = d.SaleDate
        AND x.LegalReference = d.LegalReference        
        AND x.UniqueID_ < d.UniqueID_                 
        )
        ;
```
- Dropped unused columns
```sql
select
    *
from
    `project-for-portfolio.data_cleaning_project.houses`
limit 20;


alter table
    `project-for-portfolio.data_cleaning_project.houses`
drop Column IF EXISTS
    PropertyAddress,
drop column if EXISTS
    OwnerAddress,
drop column if EXISTS
    TaxDistrict;
```