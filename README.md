# Design Patters: Medieval PHP Edition

There are plenty abstract, lengthy guides about design patterns. I decided to take another approach: imagine for a moment that PHP was not a language to write web applications, but to write a medieval life. With this in mind, let's cover some design patters that you can use with PHP. 

## Singleton 

Let's start with creating our kingdom. The kingdom will have many farmers, warriors, and craftsmen, but only one king. Thus, let's make a class to get the one king. 

```php
final class KingOfPHP
{
    // make this private so no one can create a new king
    private function __construct()
    {
    }

    public static function instance()
    {
      static $kingInstance = null;
      if ($kingInstance === null) {
        $kingInstance = new KingOfPHP();
      }
      return $kingInstance;
    }


}

$king = KingOfPHP::instance();
$sameking = KingOfPHP::instance();

var_dump($king1 == $sameking); // same king!
```

With this pattern, we make sure that we always get that instance of the one king we have, whether or not it has been created.

## Factory 

Now we want to move to creating warriors. We wrote a class for swordsmen already.

```php
class Swordsman
{
  public function getWeapon()
  {
    return 'sword';
  }
}
```

However, we realize that bowmen might also be a good idea, and might need to change this in the future. For now, when we call a warrior, we will call swordsmen. This is where the factory pattern comes in handy:

```php
class Warrior
{
  public static function summon()
  {
    return new Swordsman();
  }
}
```

Now we can summon however many warriors we want, without worrying about what the current choice for warrior is. 

```php
$warrior1 = Warrior::summon();
$warrior2 = Warrior::summon();
$warrior3 = Warrior::summon();
```

## Chain of Command

With our warriors set, we need to set their attack/defense plan. They will adapt in the battlefield, and have a set of techniques in mind that they will need to recall when fighting. 

```php
interface Technique
{
  public function perform($name);
}

class SwordTechnique implements Technique
{
  public function perform($name)
  {
    if ($name != 'doubleCut') {
      return 'false';
    }
    echo 'cut cut!';
  }
}

class ShieldTechnique implements Technique
{
  public function perform($name)
  {
    if ($name != 'parry') {
      return false;
    }
    echo 'KLANG!';
  }
}
```
But it will be up to each warrior's memory to recall his training.

```php
class FightingMemory
{
  private $techniques;

  public function addTechnique($tech)
  {
    $this->techniques[] = $tech;
  }

  public function recallTechnique($name) {
    foreach($this->techniques as $t) {
      $t->perform($name);
    }
  }
}
``` 
Now you can learn and recall those techniques. 

```php
$warriorMemory = new FightingMemory();
$warriorMemory->addTechnique(new SwordTechnique());
$warriorMemory->addTechnique(new ShieldTechnique());
$warriorMemory->recallTechnique('parry');
$warriorMemory->recallTechnique('doubleCut');
$warriorMemory->recallTechnique('doubleCut');
```

## Observer

The king now decides to set watchtowers around villages near strategic points so that they can report back in case of a foreign invasion. 

```php
interface Watchtower
{
  public function report($message);
}

interface StrategicPoint
{
  public function addWatchtower($watchtower);
}
```
Let's look at the code of one such village.

```php
class CoastalVillage implements StrategicPoint
{
    private $watchtowers = [];

    public function addWatchtower($watchtower)
    {
      $this->watchtowers[] = $watchtower;
    }

    public function callForAlarm($message)
    {
      foreach($this->watchtowers as $w) {
        $w->report($message);
      }
    }
}
```

...and the different watchtowers. 

```php 
class HornWatchtower implements Watchtower
{
  public function report($message)
  {
    echo $message . ', sound the horns!!!!';
  }
}

class BellWatchtower implements Watchtower
{
  public function report($message)
  {
    echo $message . ', sound the bells!!!';
  }
}
```

Finally, let's see how they would respond if they saw incoming boats. 


```php
$coastalVillage1 = new CoastalVillage();
$coastalVillage1->addWatchtower(new HornWatchtower());
$coastalVillage1->addWatchtower(new BellWatchtower());
$coastalVillage1->callForAlarm('I see ten boats');
```

