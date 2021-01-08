---
title: GMTK Game Jam
sample: Game Dev Competition
date: 2021-01-05 10:00:00 +07:00
categories: [Competitions, Game Dev] 
tags: [game development, design, competition, game jam, rust, programming, coding, game design, open source]
description: My entry for the 2019 Game Makers Tool Kit game jam, one of the largest game jams of the year
---
# Overview
Every year GMTK(Game Makers Tool Kit) hosts on of the largest game jams on [itch.io](https://itch.io). You have 42 hours to design and make a game fitting the selected theme. 

I participated in the 2019 game jam that had the theme `only one`. I mostly entered for fun and for the experience. I didn't enter with a team had family stuff going on that weekend so didn't put an extreme amount of time into the event. Also, for a lot of these game jams people generally start with a prefab base layout of their game and modify it to fit the topic at hand. I didn't do this and just used a very minimal opensource framework [quicksilver](https://github.com/ryanisaacg/quicksilver) that I contributed too. 

Overall though it was a fun experience getting something working under such a short time frame even if it wasn't very polished.

You can download the game [here](https://github.com/dallasc/systemscritical). You have to have rust installed then just run cargo run. Right now there aren't any binaries available but it should work for any Operating system.

## Idea
Since I didn't have much time decided to keep the game simple. I opted to make a 2d game similar to the classic game asteroids. To make the game more interesting and to fit the theme of the game jam you play as a space ship but your engines are damaged. You only have enough power to use one of the ships systems at a time. The 3 systems for the ship are `engines`, `weapons`, and `radar`. You switch between these with 1,2,3 on your keyboard. W activates whatever system is active. Also you can always turn to clockwise or counter-clockwise using A and D. 

When `engines` are active and you press W it moves your ship forward in the direction you are moving. Even if you stop pressing W you still preserve momentum in the direction you were last traveling but slowly lose it. `weapons` fires a laser in front of you like in the traditional game. `radar` is pretty unique. When the game starts everything is black. Pressing W when radar is active fires out a circular wave with you as it's center allowing you to see asteroid locations as it passes over them. What this means is if you don't use the radar you might hit an asteroid and not even know what killed you. 

Additionally there is a random purple wormhole that spawns on each level. You have to reach it in order to move onto the next level. You can destroy the asteroids or try to avoid the them. 

Additionally there were suppose to be Enemy ships appear but I ran out of time and didn't get to fully implement them. 

## Implementation
You can see the source on [github](https://github.com/dallasc/systemscritical). Everything is stored in one file in `src/main.rs`, normally I would organize my code a bit cleaner but it's a relatively small file just 700 lines total with over half that being boilerplate code to get things set up with the game engine.

### Framework
I used [quicksilver](https://github.com/ryanisaacg/quicksilver) at the time. I was pretty familiar with it and had made some contributions to it so felt comfortable working with it. It is now unmaintained thought, this was awhile ago though when rust's game dev scene was still in it's baby stages and there were a lot of different frameworks for 2d games. Now it seems most of the community has settled around using [bevy](https://github.com/bevyengine/bevy/) and if I were to start another game project I would most likely end up using it too.

### Code
I'll give a basic rundown of the main components of the code. You can see the whole thing on github it's commented and is pretty straight forward since its such a small game.

First up we have our basic unit a `Vector` and `Point`. Most game engines have these built in with the helper functions already there but I had to make them myself. One to make a vector from an angle and another to generate random vectors for spawning locations and speeds.
``` Rust
type Point2 = geom::Vector;
type Vector2 = geom::Vector;

fn vec_from_angle(angle: f32) -> Vector2 {
    let vx = angle.sin();
    let vy = angle.cos();
    Vector2::new(vx, vy)
}

fn random_vec(max_magnitude: f32) -> Vector2 {
    let angle = rand::random::<f32>() * 2.0 * std::f32::consts::PI;
    let mag = rand::random::<f32>() * max_magnitude;
    vec_from_angle(angle) * (mag)
}

```

There's also no built in Entity Component System(ECS) so I choose to set up my own little janky ECS. It works for this because it's such a small game and I don't need all the helper functions a full ECS provides. I can just manually register everything with each other.

``` Rust
#[derive(Debug, PartialEq)]
enum ActorType {
    Player,
    Rock,
    Shot,
    Radar,
    Wormhole,
}

#[derive(Debug, PartialEq)]
enum Systems {
    Engines,
    Wepons,
    Radar,
}

#[derive(Debug)]
struct Actor {
    tag: ActorType,
    sys: Systems,
    pos: Point2,
    facing: f32,
    velocity: Vector2,
    ang_vel: f32,
    bbox_size: f32,
    layer: i32,
    life: f32,
}

```
As you can see we have 4 types of Actors. Player (your spaceship), Rock (the astroids), shot (your wepon lazers), Radar (the radar waves), Wormhole (the exit to the next level). There are also the 3 systems (Engine, Wepon, Radar) that we talked about in the game idea.

Each actor is one of the actor types and stores its current postion, Current dirction, Current velocity, Angular Velocity, Bounding box info, layer, and life. Life has two meanings. For shot and radar it is the amount of time it has left before it disappears and for player and rock it is the actual hit points that are left. 

Next we have to be able to initialize our Actors. They are all pretty much the same for each type and look like this:
``` Rust
fn create_player() -> Actor {
    Actor {
        tag: ActorType::Player,
        sys: Systems::Radar,
        pos: Vector2::ZERO,
        facing: 0.,
        velocity: Vector2::ZERO,
        ang_vel: 0.,
        bbox_size: PLAYER_BBOX,
        layer: 500,
        life: PLAYER_LIFE,
    }
}
```
As you can see we just create an actor with the default values. We do this for each Actor Type, nothing to fancy here.

Next we have have an additional helper function to spawn in the rocks that looks like: 
```Rust

/// Create the given number of rocks.
/// Makes sure that none of them are within the
/// given exclusion zone (nominally the player)
/// Note that this *could* create rocks outside the
/// bounds of the playing field, so it should be
/// called before `wrap_actor_position()` happens.
fn create_rocks(num: i32, exclusion: Point2, min_radius: f32, max_radius: f32) -> Vec<Actor> {
    assert!(max_radius > min_radius);
    let new_rock = |_| {
        let mut rock = create_rock();
        let r_angle = rand::random::<f32>() * 2.0 * std::f32::consts::PI;
        let r_distance = rand::random::<f32>() * (max_radius - min_radius) + min_radius;
        rock.pos = exclusion + vec_from_angle(r_angle) * r_distance;
        rock.velocity = random_vec(MAX_ROCK_VEL);
        rock
    };
    (0..num).map(new_rock).collect()
}
```
Here we create the given number of astroids depending on the level. It pretty much just makes a `Vec` containing `Rock` Actors with a random starting point and random velocity and sizes. Note that we also create an exclusion zone around the player to make sure that we spawn a rock right on top of them. 

We also do the same thing for the wormhole but we only have to make one. It's the same function pretty much though just that its velocity set to 0 so it can't move as you can see here:
``` Rust
fn create_wormholes(num: i32, exclusion: Point2, min_radius: f32, max_radius: f32) -> Vec<Actor> {
    assert!(max_radius > min_radius);
    let new_wormhole = |_| {
        let mut wormhole = create_wormhole();
        let r_angle = rand::random::<f32>() * 2.0 * std::f32::consts::PI;
        let r_distance = rand::random::<f32>() * (max_radius - min_radius) + min_radius;
        wormhole.pos = exclusion + vec_from_angle(r_angle) * r_distance;
        wormhole.velocity = random_vec(MAX_WORMHOLE_VEL);
        wormhole
    };
    (0..num).map(new_wormhole).collect()
}
```
Next we have the player input. It basically just translates A and D into angular components that increment or deincrement the players facing direction. If they are going forward then we call `player_thrust` which computes its angular velocity.
``` Rust
fn player_handle_input(actor: &mut Actor, input: &InputState, dt: f32) {
    actor.facing += dt * PLAYER_TURN_RATE * input.xaxis;

    if input.yaxis > 0.0 {
        player_thrust(actor, dt);
    }
}

fn player_thrust(actor: &mut Actor, dt: f32) {
    let direction_vector = vec_from_angle(actor.facing);
    let thrust_vector = direction_vector * (PLAYER_THRUST);
    actor.velocity += thrust_vector * (dt);
}
```
Next we just have to update the players position by taking the players old position and using it's new velocity and angular velocity vectors to computer its new location for the given timestep.
``` Rust

const MAX_PHYSICS_VEL: f32 = 200.0;

fn update_actor_position(actor: &mut Actor, dt: f32) {
    // Clamp the velocity to the max efficiently
    let norm_sq = actor.velocity.len2();
    if norm_sq > MAX_PHYSICS_VEL.powi(2) {
        actor.velocity = actor.velocity / norm_sq.sqrt() * MAX_PHYSICS_VEL;
    }
    let dv = actor.velocity * (dt);
    actor.pos += dv;
    actor.facing += actor.ang_vel;
}
```

We also add a little helper function to create a screen wrap effect. This allows us to go off one side of the screen and it places us on the other side of the screen.
``` Rust
fn wrap_actor_position(actor: &mut Actor, sx: f32, sy: f32) {
    // Wrap screen
    let screen_x_bounds = sx / 2.0;
    let screen_y_bounds = sy / 2.0;
    if actor.pos.x > screen_x_bounds {
        actor.pos.x -= sx;
    } else if actor.pos.x < -screen_x_bounds {
        actor.pos.x += sx;
    };
    if actor.pos.y > screen_y_bounds {
        actor.pos.y -= sy;
    } else if actor.pos.y < -screen_y_bounds {
        actor.pos.y += sy;
    }
}
```

The game is very basic in design and the assets are too. You can see them in the `static` folder on github. There are some basic images for the spaceship, lazer, and astroid. I also added some classic arcade sounds for when firing a lazer and when you hit something with either a lazer or your ship.
``` Rust
struct Assets {
    player_image: Asset<Image>,
    shot_image: Asset<Image>,
    rock_image: Asset<Image>,
    font: Asset<graphics::Font>,
    shot_sound: Asset<sound::Sound>,
    hit_sound: Asset<sound::Sound>,
}

impl Assets {
    fn new() -> quicksilver::Result<Assets> {
        let player_image = Asset::new(Image::load("player.png"));
        let shot_image = Asset::new(Image::load("shot.png"));
        let rock_image = Asset::new(Image::load("astroid.png"));
        let font = Asset::new(graphics::Font::load("DejaVuSerif.ttf"));

        let shot_sound = Asset::new(sound::Sound::load("pew.ogg"));
        let hit_sound = Asset::new(sound::Sound::load("boom.ogg"));

        Ok(Assets {
            player_image,
            shot_image,
            rock_image,
            font,
            shot_sound,
            hit_sound,
        })
    }

    fn actor_image(&mut self, actor: &Actor) -> &mut Asset<Image> {
        match actor.tag {
            ActorType::Player => &mut self.player_image,
            ActorType::Rock => &mut self.rock_image,
            ActorType::Shot => &mut self.shot_image,
            ActorType::Radar => &mut self.rock_image,
            ActorType::Wormhole => &mut self.rock_image,
        }
    }
}
```
Most of it is just boilerplate from `quicksilver` for loading an asset. Every game engine does this differently so there's not much to learn from here.

Next we have user inputs:
``` Rust
 #[derive(Debug)]
struct InputState {
    xaxis: f32,
    yaxis: f32,
    fire: bool,
    radar: bool,
}

impl Default for InputState {
    fn default() -> Self {
        InputState {
            xaxis: 0.0,
            yaxis: 0.0,
            fire: false,
            radar: false,
        }
    }
}
```
`xaxis` is when either A or D is pressed and translates to a change in the direction the ship is facing. Similarlly `yaxis` is used when the engine is turned on and adds velocity to the direction you are facing. `fire` and `radar` are both bools. If true it means you are holding them down and they continue to fire until you release the key. There is a built in cooldown to each type if you noticed earlier though. This prevents the player from spamming infinite lazers if they are holding down the button.

Next we have the overall game state. This includes things like score, all the actors on the screen, current level and some other utility stuff like shot cooldown etc.
``` Rust
struct MainState {
    player: Actor,
    shots: Vec<Actor>,
    radar: Vec<Actor>,
    rocks: Vec<Actor>,
    wormhole: Vec<Actor>,
    level: i32,
    score: i32,
    assets: Assets,
    screen_width: f32,
    screen_height: f32,
    input: InputState,
    player_shot_timeout: f32,
    player_radar_timeout: f32,
    radar_layer: i32,
}

impl MainState {
    fn new() -> quicksilver::Result<MainState> {
        print_instructions();

        let assets = Assets::new()?;
        let player = create_player();
        let rocks = create_rocks(5, player.pos, 100.0, 250.0);
        let wormhole = create_wormholes(1, player.pos, 100.0, 250.0);

        let window_size = Vector2::new(800.0, 600.0);
        let s = MainState {
            player,
            shots: Vec::new(),
            radar: Vec::new(),
            rocks,
            wormhole,
            level: 0,
            score: 0,
            assets,
            screen_width: window_size.x,
            screen_height: window_size.y,
            input: InputState::default(),
            player_shot_timeout: 0.0,
            player_radar_timeout: 0.0,
            radar_layer: 0,
        };

        Ok(s)
    }

    fn reset(&mut self) {
        self.player = create_player();
        self.shots = Vec::new();
        self.radar = Vec::new();
        self.rocks = create_rocks(5, self.player.pos, 100.0, 250.0);
        self.wormhole = create_wormholes(1, self.player.pos, 100.0, 250.0);
        self.level = 0;
        self.score = 0;
        self.input = InputState::default();
        self.player_shot_timeout = 0.0;
        self.player_radar_timeout = 0.0;
        self.radar_layer = 0;
    }

    fn fire_player_shot(&mut self) {
        self.player_shot_timeout = PLAYER_SHOT_TIME;

        let player = &self.player;
        let mut shot = create_shot();
        shot.pos = player.pos;
        shot.facing = player.facing;
        let direction = vec_from_angle(shot.facing);
        shot.velocity.x = SHOT_SPEED * direction.x;
        shot.velocity.y = SHOT_SPEED * direction.y;

        self.shots.push(shot);

        let _ = self.assets.shot_sound.execute(|s| s.play());
    }

    fn fire_player_radar(&mut self) {
        self.player_radar_timeout = PLAYER_RADAR_TIME;

        let player = &self.player;
        let mut radar = create_radar(self.radar_layer);
        radar.pos = player.pos;
        self.radar_layer = self.radar_layer + 2;

        self.radar.push(radar);

        let _ = self.assets.shot_sound.execute(|s| s.play());
    }

    fn clear_dead_stuff(&mut self) {
        self.shots.retain(|s| s.life > 0.0);
        self.rocks.retain(|r| r.life > 0.0);
        self.radar.retain(|r| r.life > 0.0);
        self.wormhole.retain(|w| w.life > 0.0);
        if self.radar.len() == 0 {
            self.radar_layer = 0
        }
    }

    fn handle_collisions(&mut self) {
        for rock in &mut self.rocks {
            let pdistance = rock.pos - self.player.pos;
            if pdistance.len() < (self.player.bbox_size + rock.bbox_size) {
                self.player.life = 0.0;
            }
            for shot in &mut self.shots {
                let distance = shot.pos - rock.pos;
                if distance.len() < (shot.bbox_size + rock.bbox_size) {
                    shot.life = 0.0;
                    rock.life = 0.0;
                    self.score += 1;

                    let _ = self.assets.hit_sound.execute(|s| s.play());
                }
            }
        }
        for wormhole in &mut self.wormhole {
            let pdistance = wormhole.pos - self.player.pos;
            if pdistance.len() < (self.player.bbox_size + wormhole.bbox_size) {
                wormhole.life = 0.;
            }
        }
    }

    fn check_for_level_end(&mut self) {
        if self.wormhole.is_empty() {
            self.score += 10;
            self.level += 1;
            self.wormhole = create_wormholes(1, self.player.pos, 100.0, 250.0);
            self.rocks = create_rocks(self.level * 2 + 5, self.player.pos, 100.0, 250.0);
        }
    }
}
```
This is a lot to take in but most of it is just initializing things and helper function. `new()` and `reset()` just start a new game or clean up the game when you die and start over. `fire_player_shot` and `fire_player_radar` just create a new actor for the players shot/radar as it comes onto the screen. `handle_collision` just checks if a lazer hit a rock/yourself hit a rock or if you made it to the wormhole. `clear_dead_stuff` checks if the rock has any life left and removes it if it doesn't. Lastly `check_for_level_end` does exactly what it sounds like and gives you points for the level and moves you onto the next level.
 
Lastly we have a big section coming up. It may look confusing but its just `quicksilver` boilerplate for drawing things on screen and handling the update cycle for the game time-step. I'm not going to explain this in detail because it's only for quicksilver and doesn't really translate to other game engines. Quicksilver is no unmaintained and if I were to make this at the current date I would use `Bevy`. It is commented pretty well if you did want to look though.
``` Rust

impl State for MainState {
    fn new() -> quicksilver::Result<Self> {
        MainState::new()
    }
    
    fn update(&mut self, _window: &mut Window) -> quicksilver::Result<()> {
        const DESIRED_FPS: u32 = 60;
        let seconds = 1.0 / (DESIRED_FPS as f32);

        // Update the player state based on the user input.
        player_handle_input(&mut self.player, &self.input, seconds);
        self.player_shot_timeout -= seconds;
        if self.input.fire && self.player_shot_timeout < 0.0 {
            self.fire_player_shot();
        }
        self.player_radar_timeout -= seconds;
        if self.input.radar && self.player_radar_timeout < 0.0 {
            self.fire_player_radar();
        }

        // Update the physics for all actors.
        // First the player...
        update_actor_position(&mut self.player, seconds);
        wrap_actor_position(
            &mut self.player,
            self.screen_width as f32,
            self.screen_height as f32,
        );

        // Then the shots...
        for act in &mut self.shots {
            update_actor_position(act, seconds);
            wrap_actor_position(act, self.screen_width as f32, self.screen_height as f32);
            handle_timed_life(act, seconds);
        }

        // And radar
        for act in &mut self.radar {
            handle_timed_life(act, seconds);
        }

        // And finally the rocks.
        for act in &mut self.rocks {
            update_actor_position(act, seconds);
            wrap_actor_position(act, self.screen_width as f32, self.screen_height as f32);
        }

        // Handle the results of things moving:
        // collision detection, object death, and if
        // we have killed all the rocks in the level,
        // spawn more of them.
        self.handle_collisions();

        self.clear_dead_stuff();

        // self.check_for_level_respawn();
        self.check_for_level_end();
        // Finally we check for our end state.
        // I want to have a nice death screen eventually,
        // but for now we just quit.
        if self.player.life <= 0.0 {
            println!("Your score was {}", self.score);
            println!("Your level was {}", self.level);
            println!("Try Again");
            MainState::reset(self);
        }

        Ok(())
    }

    fn event(&mut self, event: &Event, _window: &mut Window) -> quicksilver::Result<()> {
        match event {
            // Buttons pressed
            Event::Key(Key::Key1, ButtonState::Pressed) => {
                self.player.sys = Systems::Engines;
            }
            Event::Key(Key::Key2, ButtonState::Pressed) => {
                self.player.sys = Systems::Wepons;
            }
            Event::Key(Key::Key3, ButtonState::Pressed) => {
                self.player.sys = Systems::Radar;
            }
            Event::Key(Key::W, ButtonState::Pressed) => {
                if self.player.sys == Systems::Radar {
                    self.input.radar = true;
                } else if self.player.sys == Systems::Wepons {
                    self.input.fire = true;
                } else {
                    self.input.yaxis = 1.0;
                }
            }
            Event::Key(Key::A, ButtonState::Pressed) => {
                self.input.xaxis = -1.0;
            }
            Event::Key(Key::D, ButtonState::Pressed) => {
                self.input.xaxis = 1.0;
            }
            Event::Key(Key::Escape, ButtonState::Pressed) => {
                std::process::exit(0);
            }
            // Buttons released
            Event::Key(Key::W, ButtonState::Released) => {
                self.input.yaxis = 0.0;
                self.input.fire = false;
                self.input.radar = false;
            }
            Event::Key(Key::A, ButtonState::Released) => {
                self.input.xaxis = 0.0;
            }
            Event::Key(Key::D, ButtonState::Released) => {
                self.input.xaxis = 0.0;
            }
            _ => (), // Do nothing
        }
        Ok(())
    }

    fn draw(&mut self, window: &mut Window) -> quicksilver::Result<()> {
        // Clear the screen...
        window.clear(Color::BLACK)?;

        // Loop over all objects drawing them...
        {
            let assets = &mut self.assets;
            let coords = (self.screen_width, self.screen_height);

            let p = &self.player;
            draw_actor(assets, window, p, coords)?;

            for s in &self.shots {
                draw_actor(assets, window, s, coords)?;
            }

            for r in &self.rocks {
                draw_actor(assets, window, r, coords)?;
            }

            for r in &self.radar {
                draw_actor(assets, window, r, coords)?;
            }

            for w in &self.wormhole {
                draw_actor(assets, window, w, coords)?;
            }
        }

        // And draw the GUI elements in the right places.
        let level_dest = Point2::new(100.0, 10.0);
        let score_dest = Point2::new(300.0, 10.0);

        let level_str = format!("Level: {}", self.level);
        let score_str = format!("Score: {}", self.score);

        self.assets.font.execute(|f| {
            let style = FontStyle::new(24.0, Color::WHITE);
            let text = f.render(&level_str, &style)?;
            window.draw(&text.area().with_center(level_dest), Background::Img(&text));

            let text = f.render(&score_str, &style)?;
            window.draw(&text.area().with_center(score_dest), Background::Img(&text));

            Ok(())
        })?;

        Ok(())
    }
}

pub fn main() -> quicksilver::Result<()> {
    run::<MainState>("Systems Critical", Vector::new(800, 600),
        Settings::default()
    );
    Ok(())
}
```
And that’s it!
## Postmortem
There were a lot of things I ran out of time for.

The goal was for the player to be making a great escape in a damaged ship through an asteroid field while being chased by hostile enemy ships.

Unfortunately being a solo developer with other things going on and the event being only 2 days, I wasn't able to finish everything that I wanted.

The main thing that I ran out of time to do was getting the enemy ship AI working. I had a basic version completed but it didn't attempt to dodge asteroids and it was only 1 type of enemy ship. I opted not to include it in the finial game because I felt that it made the game feel more sloppy than without.

I also wanted to design larger levels to play but had to settle for a basic asteroids game like level with randomly generated asteroids.

Lastly I didn't have time to add better menu's and game ending screen which would have made the game feel more polished.

## Closing thoughts
Competitions are fun. I want to do another one in the future maybe this year. I'll look into using [Bevy](https://github.com/bevyengine/bevy) it looks quite promising and has a pretty high rate of development. Getting a project base setup would save a lot of time during the jam and is what most people do, it makes sense because then you can spend the time working on game specific features instead of setting up the general stuff like loading screens, menus, UI, controls, etc. Lastly, I think it would be best to work with other people especially a designer. It's to much to do everything yourself in that amount of time and while simple graphics can be fun I really think that having working with a designer would make the final product look much more polished.
