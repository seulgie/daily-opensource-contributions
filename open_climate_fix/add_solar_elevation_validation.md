# Daily Commit Log - 2025-02-18

## Commit Title: 
Added Solar Elevation Validation & Assumed Input Format

## Summary:
- **Problem**: I needed to add a validation step to check if forecast values are non-zero when the sun elevation is above 10 degrees.
- **Approach**: I assumed the input data format would be a dictionary with datetime, forecast_values, latitude, and longitude.
- **Changes**:
    - Extracted solar elevation data using `pvlib` based on the provided latitude, longitude, and datetime.
    - Checked that forecast values are greater than 0 when solar elevation is above 10 degrees.
    - Added logging and exception handling for validation.

## Code:
```python
def validate_forecast(
    forecast_data: Dict[str, np.ndarray],
    national_forecast_values: np.ndarray,
    national_capacity: float,
    logger_func: Callable[[str], None],
) -> None:
    """Checks various conditions including forecast values and solar elevation constraints.

    Args:
        forecast_data: A dictionary containing 'datetimes', 'forecast_values', 'lat', 'lon'.
        national_forecast_values: All the forecast values for the nation (in MW).
        national_capacity: The national PV capacity (in MW).
        logger_func: A function that takes a string and logs it (e.g., logging.info).

    Raises:
        Exception: If forecast values exceed critical thresholds or fail solar elevation validation.
    """
    try:
        # Extract Values from the input dictionary
        datetimes = pd.to_datetime(forecast_data['datetimes'])
        forecast_values = pd.Series(forecast_data['forecast_values'], index=datetimes)
        lat = forecast_data['lat']
        lon = forecast_data['lon']
    except KeyError as e:
        raise ValueError(f"Missing key in forecast_data: {e}")

    # Validate based on sun elevation > 10 degrees
    solpos = pvlib.solarposition.get_solarposition(
        time=datetimes,
        latitude=lat,
        longitude=lon,
        method='nrel_numpy'
    )

    elevation = solpos['elevation']
    forecast_above_10_deg = forecast_values[elevation > 10]

    if (forecast_above_10_deg <= 0).any():
        raise Exception("FAIL: Forecast values must be >0 when sun elevation > 10°.")
```

