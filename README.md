# Collect sytem statistics inside your Laravel app

The ```vanquynguyen/laravel-system-statistics``` package provides easy to use functions to collect the system statistics of your app. The Package collects all data in the ```system_statistics``` table.

[![Latest Version on Packagist](http://img.shields.io/static/v1?label=packagist&message=v.1.0.0&color=%3CCOLOR%3E)](https://packagist.org/packages/vanquynguyen/laravel-system-statistics)

# Installation
You can install the package via composer:
```
composer require vanquynguyen/laravel-system-statistics
```

The package will automatically register itself.

You can publish the migration with:

```
php artisan vendor:publish --provider="Vanquy\SystemStatistics\SystemStatisticServiceProvider" --tag="migrations"
```

After publishing the migration you can create the system_statistics table by running the migrations:

```
php artisan migrate
```

You can optionally publish the config file with:

```
php artisan vendor:publish --provider="Vanquy\SystemStatistics\SystemStatisticServiceProvider" --tag="config"
```
You can optionally publish the command file to collect data with:

```
php artisan vendor:publish --provider="Vanquy\SystemStatistics\SystemStatisticServiceProvider" --tag="commands"
```
# How to use?
### Models
You can retrieve all data using the ```Vanquy\SystemStatistics\Models\SystemStatistic``` model.
```php
  use Vanquy\SystemStatistics\Models\SystemStatistic;

  SystemStatistic::all();
```
Or create App\Models
```php

  namespace App\Models;

  use Vanquy\SystemStatistics\Models\SystemStatistic;

  class StatisticModel extends SystemStatistic
  {
    //
  }
```
### Available functions
You can use some available functions in:
```Vanquy\SystemStatistics\Components\Collector``` and ```Vanquy\SystemStatistics\Components\Repository```
### Commands
#### Collect data with date
```php
  namespace App\Console\Commands;

  use Vanquy\SystemStatistics\Components\Collector;

  /**
   * Execute the console command.
   *
   * @return mixed
   */
  public function handle(Collector $statsCollector)
  {
      $dateArg = $this->option('date');
      $date = $dateArg !== null ? Carbon::createFromFormat('Y-m-d', $dateArg) : Carbon::now()->subDay(1);

      $this->deleteOldStatistics($date);
      $queries = [];
      $statistics = $statsCollector->forDate($queries, $date);

      $data = $statistics->map(function ($item, $key) use ($date) {
          return [
              'type' => $key,
              'data' => $item,
              'date' => $date,
          ];
      });

      SystemStatistic::insert($data->toArray());
  }
```
You can fill array queries to $queries variable.
Ex: 
```php
  namespace App\Console\Commands;

  use Vanquy\SystemStatistics\Components\Collector;
  
  public function handle(Collector $statsCollector)
  {
    $queries = [
      [
          'type' => 'total_users',
          'query' => $statsCollector->countTotalSystem('users'),
      ],
    ];
    ...
  }
```
#### Send daily report
```php
  namespace App\Console\Commands;

  use Vanquy\SystemStatistics\Components\Repository

  /**
   * Execute the console command.
   *
   * @return mixed
   */
  public function handle(Repository $statsRepository)
  {
      $currentDate = Carbon::now()->subDay(1);
      $lastDate = Carbon::now()->subDay(2);
      $types = [];

      $dataStatistics = $statsRepository->getStatisticsForReport($types, $currentDate, $lastDate);

      return $dataStatistics;
  }
```

You can fill array types to $types variable.
Ex:
```php
$type = ['total_users', 'total_new_users'];
.....
```
## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
