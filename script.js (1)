
/* =========================================
   CLOUDORA WEATHER APP
   Real weather using Open-Meteo API
========================================= */


/* =========================================
   1. SELECT HTML ELEMENTS
========================================= */

const searchForm = document.getElementById("searchForm");
const cityInput = document.getElementById("cityInput");
const searchButton = document.getElementById("searchButton");
const searchButtonText = document.getElementById("searchButtonText");
const statusMessage = document.getElementById("statusMessage");

const cityName = document.getElementById("cityName");
const currentDate = document.getElementById("currentDate");
const mainWeatherIcon = document.getElementById("mainWeatherIcon");
const temperature = document.getElementById("temperature");
const weatherDescription = document.getElementById("weatherDescription");
const feelsLike = document.getElementById("feelsLike");

const todayHigh = document.getElementById("todayHigh");
const todayLow = document.getElementById("todayLow");

const humidity = document.getElementById("humidity");
const windSpeed = document.getElementById("windSpeed");
const cloudCover = document.getElementById("cloudCover");
const rainAmount = document.getElementById("rainAmount");

const hourlyForecast = document.getElementById("hourlyForecast");
const dailyForecast = document.getElementById("dailyForecast");


/* =========================================
   2. WEATHER CODE INFORMATION

   Open-Meteo returns numerical weather codes.
   We convert those codes into descriptions
   and weather icons.
========================================= */

function getWeatherInfo(code, isDay = true) {

  if (code === 0) {
    return {
      description: "Clear sky",
      icon: isDay ? "☀️" : "🌙"
    };
  }

  if (code === 1) {
    return {
      description: "Mainly clear",
      icon: isDay ? "🌤️" : "🌙"
    };
  }

  if (code === 2) {
    return {
      description: "Partly cloudy",
      icon: "⛅"
    };
  }

  if (code === 3) {
    return {
      description: "Overcast",
      icon: "☁️"
    };
  }

  if ([45, 48].includes(code)) {
    return {
      description: "Foggy",
      icon: "🌫️"
    };
  }

  if ([51, 53, 55].includes(code)) {
    return {
      description: "Drizzle",
      icon: "🌦️"
    };
  }

  if ([56, 57].includes(code)) {
    return {
      description: "Freezing drizzle",
      icon: "🌧️"
    };
  }

  if ([61, 63, 65].includes(code)) {
    return {
      description: "Rain",
      icon: "🌧️"
    };
  }

  if ([66, 67].includes(code)) {
    return {
      description: "Freezing rain",
      icon: "🌧️"
    };
  }

  if ([71, 73, 75, 77].includes(code)) {
    return {
      description: "Snowfall",
      icon: "❄️"
    };
  }

  if ([80, 81, 82].includes(code)) {
    return {
      description: "Rain showers",
      icon: "🌦️"
    };
  }

  if ([85, 86].includes(code)) {
    return {
      description: "Snow showers",
      icon: "🌨️"
    };
  }

  if (code === 95) {
    return {
      description: "Thunderstorm",
      icon: "⛈️"
    };
  }

  if ([96, 99].includes(code)) {
    return {
      description: "Thunderstorm with hail",
      icon: "⛈️"
    };
  }

  return {
    description: "Weather unavailable",
    icon: "🌡️"
  };
}


/* =========================================
   3. FORMAT DATE AND TIME
========================================= */

function formatDate(dateString) {

  // Add a time to date-only values so that
  // the displayed date doesn't shift.
  const date = new Date(dateString + "T12:00:00");

  return date.toLocaleDateString("en-IN", {
    weekday: "long",
    day: "numeric",
    month: "long"
  });
}


function formatHour(dateString) {

  // Weather API times are local city times.
  // Read the hour directly from the returned text.
  const match = dateString.match(/T(\d{2}):(\d{2})/);

  if (!match) {
    return "--";
  }

  const hour = Number(match[1]);
  const minutes = match[2];

  const suffix = hour >= 12 ? "PM" : "AM";
  const displayHour = hour % 12 || 12;

  return minutes === "00"
    ? `${displayHour} ${suffix}`
    : `${displayHour}:${minutes} ${suffix}`;
}


function formatDay(dateString, index) {

  if (index === 0) {
    return "Today";
  }

  if (index === 1) {
    return "Tomorrow";
  }

  const date = new Date(dateString + "T12:00:00");

  return date.toLocaleDateString("en-IN", {
    weekday: "long"
  });
}


/* =========================================
   4. SHOW STATUS MESSAGES
========================================= */

function showStatus(message, isError = false) {

  statusMessage.textContent = message;

  statusMessage.style.color = isError
    ? "#c04466"
    : "#817c9a";
}


/* =========================================
   5. FIND A CITY

   First API:
   Open-Meteo Geocoding API

   It returns the city's coordinates.
========================================= */

async function getCityCoordinates(city) {

  const url =
    "https://geocoding-api.open-meteo.com/v1/search" +
    `?name=${encodeURIComponent(city)}` +
    "&count=10&language=en&format=json";

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(
      "Could not connect to the city search service."
    );
  }

  const data = await response.json();

  if (!data.results || data.results.length === 0) {
    throw new Error(
      "City not found. Please check the spelling."
    );
  }

  // Prefer an exact city-name match when possible.
  const exactMatch = data.results.find(
    place => place.name.toLowerCase() === city.toLowerCase()
  );

  return exactMatch || data.results[0];
}


/* =========================================
   6. GET WEATHER DATA

   Second API:
   Open-Meteo Forecast API
========================================= */

async function getWeatherData(latitude, longitude) {

  const url =
    "https://api.open-meteo.com/v1/forecast" +
    `?latitude=${latitude}` +
    `&longitude=${longitude}` +
    "&current=temperature_2m,relative_humidity_2m," +
    "apparent_temperature,is_day,precipitation," +
    "weather_code,cloud_cover,wind_speed_10m" +
    "&hourly=temperature_2m,weather_code,is_day" +
    "&daily=weather_code,temperature_2m_max," +
    "temperature_2m_min,precipitation_sum" +
    "&forecast_days=5" +
    "&timezone=auto";

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(
      "Could not retrieve weather data. Please try again."
    );
  }

  const data = await response.json();

  if (!data.current || !data.daily || !data.hourly) {
    throw new Error(
      "Weather data is incomplete. Please try again."
    );
  }

  return data;
}


/* =========================================
   7. UPDATE THE MAIN WEATHER CARD
========================================= */

function updateCurrentWeather(place, data) {

  const current = data.current;
  const daily = data.daily;

  cityName.textContent = place.name;

  // Include the region/country when available.
  if (place.admin1 || place.country) {

    const locationParts = [
      place.admin1,
      place.country
    ].filter(Boolean);

    // Keep the city heading clean and readable.
    cityName.textContent = place.name;

    showStatus(
      `Showing weather for ${place.name}, ${locationParts.join(", ")}.`
    );
  } else {
    showStatus(`Showing weather for ${place.name}.`);
  }

  currentDate.textContent = formatDate(current.time);

  const weather = getWeatherInfo(
    current.weather_code,
    current.is_day === 1
  );

  mainWeatherIcon.textContent = weather.icon;

  temperature.textContent =
    Math.round(current.temperature_2m);

  weatherDescription.textContent =
    weather.description;

  feelsLike.textContent =
    Math.round(current.apparent_temperature);

  todayHigh.textContent =
    `${Math.round(daily.temperature_2m_max[0])}°`;

  todayLow.textContent =
    `${Math.round(daily.temperature_2m_min[0])}°`;
}


/* =========================================
   8. UPDATE WEATHER HIGHLIGHTS
========================================= */

function updateHighlights(data) {

  const current = data.current;
  const daily = data.daily;

  humidity.textContent =
    current.relative_humidity_2m ?? "--";

  windSpeed.textContent =
    Math.round(current.wind_speed_10m ?? 0);

  cloudCover.textContent =
    current.cloud_cover ?? "--";

  rainAmount.textContent =
    Number(daily.precipitation_sum[0] ?? 0).toFixed(1);
}


/* =========================================
   9. CREATE HOURLY FORECAST
========================================= */

function updateHourlyForecast(data) {

  const hourly = data.hourly;
  const currentTime = data.current.time;

  // Find the current hour in the city's forecast.
  let startIndex = hourly.time.findIndex(
    time => time >= currentTime
  );

  if (startIndex === -1) {
    startIndex = 0;
  }

  // Show the next eight forecast hours.
  const hoursToShow = hourly.time.slice(
    startIndex,
    startIndex + 8
  );

  if (hoursToShow.length === 0) {
    hourlyForecast.innerHTML =
      '<p class="empty-message">Hourly forecast unavailable.</p>';

    return;
  }

  hourlyForecast.innerHTML = hoursToShow.map(
    (time, index) => {

      const i = startIndex + index;

      const info = getWeatherInfo(
        hourly.weather_code[i],
        hourly.is_day[i] === 1
      );

      const temp = Math.round(
        hourly.temperature_2m[i]
      );

      const timeLabel = index === 0
        ? "Now"
        : formatHour(time);

      return `
        <article class="hour-card">
          <span class="hour-time">${timeLabel}</span>
          <span class="hour-icon">${info.icon}</span>
          <span class="hour-temp">${temp}°</span>
          <span class="hour-condition">
            ${info.description}
          </span>
        </article>
      `;
    }
  ).join("");
}


/* =========================================
   10. CREATE FIVE-DAY FORECAST
========================================= */

function updateDailyForecast(data) {

  const daily = data.daily;

  dailyForecast.innerHTML = daily.time.map(
    (date, index) => {

      const info = getWeatherInfo(
        daily.weather_code[index],
        true
      );

      const high = Math.round(
        daily.temperature_2m_max[index]
      );

      const low = Math.round(
        daily.temperature_2m_min[index]
      );

      return `
        <article class="day-row">
          <div>
            <p class="day-name">
              ${formatDay(date, index)}
            </p>
            <p class="day-date">
              ${formatDate(date)}
            </p>
          </div>

          <span class="day-icon">${info.icon}</span>

          <div class="day-temperatures">
            <span>${high}°</span>
            <span class="low">${low}°</span>
          </div>
        </article>
      `;
    }
  ).join("");
}


/* =========================================
   11. LOAD A CITY'S WEATHER
========================================= */

async function loadWeather(city) {

  const trimmedCity = city.trim();

  if (!trimmedCity) {
    showStatus(
      "Please enter a city name before searching.",
      true
    );

    cityInput.focus();
    return;
  }

  searchButton.disabled = true;
  searchButtonText.textContent = "Loading...";
  searchForm.classList.add("loading");

  showStatus(`Finding weather for ${trimmedCity}...`);

  try {

    // Step 1: Find the city.
    const place = await getCityCoordinates(
      trimmedCity
    );

    // Step 2: Fetch weather for that city.
    const data = await getWeatherData(
      place.latitude,
      place.longitude
    );

    // Step 3: Update every part of the app.
    updateCurrentWeather(place, data);
    updateHighlights(data);
    updateHourlyForecast(data);
    updateDailyForecast(data);

    // Remember the last successful city
    // for the current page session.
    cityInput.value = place.name;

  } catch (error) {

    console.error("Weather app error:", error);

    showStatus(
      error.message ||
      "Something went wrong. Check your internet and try again.",
      true
    );

  } finally {

    searchButton.disabled = false;
    searchButtonText.textContent = "Search";
    searchForm.classList.remove("loading");
  }
}


/* =========================================
   12. SEARCH BUTTON EVENT
========================================= */

searchForm.addEventListener("submit", function(event) {

  // Prevent the page from refreshing.
  event.preventDefault();

  const city = cityInput.value;

  loadWeather(city);
});


/* =========================================
   13. LOAD A DEFAULT CITY
========================================= */

// Start with Delhi so the app displays
// real weather immediately when opened.

loadWeather("Delhi");
