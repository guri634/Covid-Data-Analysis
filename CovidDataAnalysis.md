# Covid Data Analsis

We have used two csv files for this Analysis.

[covid_deaths](https://github.com/guri634/SQL-Projects/blob/main/CovidDeaths.csv) and 
[covid_vaccinations](https://github.com/guri634/SQL-Projects/blob/main/CovidVaccinations.csv).

Download the sql excutable file from [here](https://github.com/guri634/Covid-Data-Analysis/blob/main/CovidDataAnalysis.sql) for MySQL Database.

**Visualize the complete data**
```
select *
from coviddeaths
order by 3;
```

**Querying the data where continent is not _null_**
```
select *
from coviddeaths
where continent is not null
order by 3;
```


**Select Data that we are going to be using**
```
select location, date, total_cases, new_cases, total_deaths, population
from coviddeaths
where continent is not null
order by 1;
```

**Looking at Total Cases vs Total Deaths (Death Percentage i.e, Shows likelihood of dying if you contract covid in your country**
```
select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
from coviddeaths
where location = 'India' and continent is not null
order by DeathPercentage desc;
```

**Looking at Total Cases vs Population i.e, Shows what pecentage of population get covid**
```
select location, date, total_cases, population, (total_cases/population)*100 as PercentPopulationInfected
from coviddeaths
where location = 'India' and continent is not null
order by PercentPopulationInfected desc;
```

**Countries with Highest Infection Rate compared to Population**
```
select location, population, max(total_cases) as HighestInfectionCount,
	   max((total_cases/population)*100) as PercentPopulationInfected
from coviddeaths
where continent is not null
group by location, population
order by PercentPopulationInfected desc;
```

**Showing Countries with Highest Death count per Population**
```
select location, max(cast(total_deaths as signed)) as TotalDeathCount
from coviddeaths
where continent is not null
group by location
order by TotalDeathCount desc;
```
--------------------------------------------------------------
**Let's break things down by continent**

**Showing the continents with the hightest Death Counts**
```
select continent, max(cast(total_deaths as signed)) as TotalDeathCount
from coviddeaths
where continent is not null
group by continent
order by TotalDeathCount desc;
```

**Global Numbers**
```
select sum(new_cases) as TotalCases, sum(new_deaths) as TotalDeaths,
	   sum(new_deaths)/sum(new_cases)*100 as DeathPercentage
from coviddeaths
where continent is not null
-- group by date
order by 3 desc;
```

**Looking at Total Population vs Vaccinations**
```
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
	   sum(new_vaccinations) over (partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
--        RollingPeopleVaccinated/dea.population*100  (returns an error, so we use CTE)
from coviddeaths dea
join covidvaccinations vac
	on dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null
order by 2,3;
```

**Use CTE**
```
with PopvsVac (Continent, location, date, population,new_vaccinations, RollingPeopleVaccinated)
as (select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
	       sum(new_vaccinations) over (partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
from coviddeaths dea
join covidvaccinations vac
	on dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null
)
select *, RollingPeopleVaccinated/population*100 PercentageVaccinated
from PopvsVac;
```

**Temp Table**

```
drop table if exists PercentPopulationVaccinated;
```
```
create temporary table PercentPopulationVaccinated
(
Continent varchar(255),
Location varchar(255),
Date varchar(50),
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
);
```

```
insert into PercentPopulationVaccinated
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
	   sum(new_vaccinations) over (partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
from coviddeaths dea
join covidvaccinations vac
	on dea.location = vac.location
    and dea.date = vac.date;
```

```
select *, (RollingPeopleVaccinated/Population)*100 PercentageVaccinated
from PercentPopulationVaccinated;
```

**Creating View to store data from later visulization**
```
create view PercentPopulationVaccinated as
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
	   sum(new_vaccinations) over (partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
from coviddeaths dea
join covidvaccinations vac
	on dea.location = vac.location
    and dea.date = vac.datepercentpopulationvaccinated
where dea.continent is not null;
```

```
select * 
from PercentPopulationVaccinated;
```
