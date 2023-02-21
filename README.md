
# Covid 19 Data Exploration

Trong dự án này tôi sẽ phân tích dữ liệu Covid 19 thực tế lấy từ trang: https://ourworldindata.org/covid-deaths.
Sau đó tôi sẽ dùng các câu lệnh SQL để phân tích tỷ lệ nhiễm bệnh và quá trình phòng ngừa tiêm vắc xin các quốc gia và vùng lãnh thổ.

[![MasterHead](https://media.nature.com/lw100/magazine-assets/d41586-021-02261-8/d41586-021-02261-8_19552098.gif)](https://rishavchanda.io)


## Khám phá dữ liệu từ bảng

Đầu tiên tôi khám phá dữ liệu và sắp xếp dữ liệu theo cột.

```bash
Select *
From PortfolioProjects..CovidDeaths
Where continent is not null 
order by 3,4
```


Xem tỷ lệ tử vong khi mắc bệnh tại các quốc gia

```bash
Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProjects..CovidDeaths
Where location like '%states%'
and continent is not null 
order by 1,2
```
Tỷ lệ dân số thế giới mắc bệnh

```bash
Select Location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From PortfolioProjects..CovidDeaths
--Where location like '%states%'
order by 1,2
```
Những quốc gia có số ca tử vong cao nhất  
```bash
Select Location, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProjects..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by Location
order by TotalDeathCount desc
```
## Phân tích dữ liệu theo lục địa
Lục địa có số ca tỷ vong cao nhất 
```bash
Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProjects..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by continent
order by TotalDeathCount desc
```
Lục địa có số ca tỷ vong cao nhất 
```bash
Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProjects..CovidDeaths
--Where location like '%states%'
Where continent is not null 
Group by continent
order by TotalDeathCount desc
```
Tỷ lệ dân số đã nhận được ít nhất một liều vắc xin
```bash
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProjects..CovidDeaths dea
Join PortfolioProjects..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3
```
Dùng bảng CTE để triển khai tính toán dựa trên các lệnh truy vấn trước
```bash
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProjects..CovidDeaths dea
Join PortfolioProjects..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac
```
Dùng Temp Table để tạo bảng tạm thời    
```bash
DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProjects..CovidDeaths dea
Join PortfolioProjects..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated
```
Tạo View để dự trữ dữ liệu cho trực quan hóa dữ liệu 
```bash
Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProjects..CovidDeaths dea
Join PortfolioProjects..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
```
## 🛠 Skills
SQL, SQL Server, Tableau

