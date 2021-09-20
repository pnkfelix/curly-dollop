---
title : "Creating and deploying a Lambda"
weight : 23
---

Rust unfortunately does not yet have first class status with tools like SAM
(serverless application model).

However, we can repurpose existing SAM templates and plug our Rust code into
them. This extended exercise will take you through the steps for doing so.

(I am indebted to Richard Boyd for walking me through this process; much of this
exercise heavily draws upon his ["Learn Rust on AWS" tutorial][rhboyd-tut].

[rhboyd-tut]: https://github.com/richardhboyd/learn-rust-on-aws/

## Install Rust

If you do not have Rust already installed (the current AL2 Cloud9 environment
does not), then you need to install it. Run the following; the first line
downloads and runs the installer. The second line ensures the associated tools
(`rustc`, `cargo`, etc) are on your `$PATH`.

```sh
curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain nightly
source $HOME/.cargo/env
```

## Get our initial project ready

Normally, if we were starting from scratch, we might use a tool like `sam init`
to walk us through the setup for our new service.

To save time, I am skipping that step.

You will clone the project from my repository, and that will start you off with
a `template.yaml` file appropriate for our exercises today.

```sh
git clone https://github.com/pnkfelix/lil-game.git
cd lil-game
```

## Note: build artifacts somewhere "special"

The default SAM configuration tries to create a zip with all the contents of the
directory. We don't need that, and we *especially* don't want it to do that with
all the intermediate build products generated during `cargo build`.

There may be a way to tell SAM to narrow the scope of what it chooses to copy.
But for our purposes today, the easiest thing to do will be to avoid putting
build products into the `lil-game` directory.

To achieve that, we use a [config file setting](https://doc.rust-lang.org/cargo/reference/config.html#buildtarget-dir).

Run this command\:

```sh
cat > .cargo/config
[build]
target-dir = "/home/ec2-user/environment/my-target"
```

Note the use of an *absolute path* for the target directory. This will be
explained down below.

Now when we do `cargo build` in this directory, it will go into a `my-target`
directory that is *not* under the `lil-game` directory. (It happens to be a
sibling of the `lil-game` directory the way I have things laid out.)


## Do a build

Lets make sure we can at least build our project.

```sh
cargo build --release
```

This will download a bunch of upstream dependencies, so it may take a while.

It also builds several binaries; the lambda we deploy will only use one of them,
`../my-target/release/bootstrap`. (We will uncover the uses for the other generated
binaries as we progress.)

## No no, do a `sam build`

In fact, `cargo build --release` does not suffice here, because that will not move the
generated `bootstrap` binary into the location where Lambda needs it.

So instead, you need to run `sam build` now. (It would have worked up above too, but it would
have hidden any errors produced from `cargo build --release.)

`sam build` will end up invoking `make`, which uses the `Makefile` in this
directory, and *that* will call `cargo build` (and also copy the `bootstrap`
binary into the correct location).

Its worth noting here that `sam build` seems to invoke `make` from a different
directory; that is why it is important to specify an absolute path in the
`.cargo/config` file. Otherwise, the run of `sam build` will create its own
`my-target` directory somewhere else, and in my experience, that directory is
not preserved in between invocations.

## Deploy

While we skipped the `sam init` step above, we will not skip the `sam deploy`
step. Why\: The `sam deploy` step generates a `samconfig.toml` file that has
settings specific to your account in it (I believe). So, I cannot save us time
by having us all share one that I put into my github repository.

Run `sam deploy --guided`.

For stack name, I'm using `lil-game-tictactoe`. You can use whatever you like.
(But it might be a good idea to include "tictactoe" in the name, to make it easy
to distinguish later.)

For the remaining settings, I used the default, except in one case\:

```
        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name [sam-app]: lil-game-tictactoe
        AWS Region [us-east-2]: 
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [y/N]: 
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]: 
        HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
        Save arguments to configuration file [Y/n]: y
        SAM configuration file [samconfig.toml]: 
        SAM configuration environment [default]: 
```

Let this run. It will spit out a bunch of output, but at the end, if it is successful, it
should end with lines that look like\:

```
CloudFormation outputs from deployed stack
---------------------------------------------------------------------------------------
Outputs
---------------------------------------------------------------------------------------
Key                 HelloWorldApi
Description         API Gateway endpoint URL for Prod stage for Hello World function
Value               https://uniqsnwflk.execute-api.us-east-2.amazonaws.com/Prod/hello/
---------------------------------------------------------------------------------------

Successfully created/updated stack - lil-game-tictactoe4 in us-east-2
```

Take note of the URL after that value. We will be using it below.

## Test

Now that we have deployed our lambda, we can test it!

### Testing with Lambda directly

One way to test our deployed service is via the AWS Lambda website.

If you go back to the AWS Console, and then select the AWS Lambda service, you
should now see the function you deployed listed there.

Click it, and then scroll down to the section with the tabs labelled "Code",
"Test", "Monitor", ..., and choose "Test"

Then, type in this JSON for the Test Event\:


```json
{
    "path" : "/n/"
}
```

Then hit the orange "Test" button in the "Test event" box.

It should succeed, and print a result like the following:

```json
{
  "body": "{\"command\":\"new-game\",\"parsed_game_state\":\"---------\",\"player\":\"X\",\"next_game_states\":null,\"selected_move\":null,\"text\":null,\"victory\":null}",
  "statusCode": "200"
}
```

Our game service is *very* simple minded. Every invocation works by modifying
the path component of the URI. So the `/n/` path, short for "new game", is how
one requests a description of a fresh game state.

### Testing with `curl`

If you take the URL printed above, and replace the end "hello/" with a command
suitable for the game service, you should see a corresponding reponse.

So, taking the above example again\:

```sh
curl https://uniqsnwflk.execute-api.us-east-2.amazonaws.com/Prod/n/
```

prints this\:

```text
{"command":"new-game","parsed_game_state":"---------","player":"X","next_game_states":null,"selected_move":null,"text":null,"victory":null}
```

### Testing with a terminal driver UI

While the above methods are useful for debugging issues that arise during
development, they do not represent an ideal end-to-end experience of your new
service.

A real consumer product would probably offer a mobile app or at least a web page
interface to the service.

But since we want a code base we can understand in under two hours, we resort to
the next best thing\: a terminal UI program.

The provided program, `tictactui`, will send HTTPS requests the same way that
`curl` does, but it knows how to use the resulting JSON and the full set of game
service commands to provide a nicer overall user experience.

You run it like this, again using the URL that you kept note of up above.

```sh
cargo run --release --bin tictactui -- https://uniqsnwflk.execute-api.us-east-2.amazonaws.com/Prod/
```

And, if things go right, it should draw the blank tictactoe board in the upper
corner of the console and query you for your choice of next command, like so\:

```text
      |     |
 -----|-----|-----
      |     |
 -----|-----|-----
      |     |
 X moves: 1 2 3 4 5 6 7 8 9
 ? 
```

(I have found that the Cloud9 terminal UI does not handle this perfectly; you
may want to download the `lil-game` repository and run the terminal application
locally. Or you might prefer to `ssh` into the Cloud9 host from your terminal,
which would also resolve this problem.)

## Development

At this point, you have seen enough that you could just make changes to the code,
rebuild via `sam build` and redeploy via `sam deploy`.

However, you might prefer to work with some of the code base more directly. For
example, it can be awkward to debug the service while it is hosted on Lambda.
(You *can* debug the Lambda hosted code, for example by emitting logging
statements, or by adding more fields to the JSON responses returned from the
Lambda..)

### Directly running the game code

As explained in earlier sections, our software architecture has a board game
service that is separate from the command driver. The service does provide a local
interface, called `local`\:

```sh
cargo run --release --bin local
```

This UI is deliberately not as polished as the `tictactui` UI. It specifically
uses a transcript style interface, printing out each interaction separately
rather than jumping around to different locations on the terminal; this will allow
you to make use of `debug!` statements from your code as you work.

It also uses a command interface that more closely matches the path interface
used by the Lambda: you type one of `n`, `l`, `r`, or `s` to get each of the
New, List, Render, or Select commands explained in an earlier section, like so\:

```text
TicTacToe
     |     |     
-----|-----|-----
     |     |     
-----|-----|-----
     |     |     

next command: [n, l, r, s] (with optional /<game>)
? l
game: "---------"
list "---------" : [(1, "X--------"), (2, "-X-------"), (3, "--X------"), (4, "---X-----"), (5, "----X----"), (6, "-----X---"), (7, "------X--"), (8, "-------X-"), (9, "--------X")]
choose a move from list above
(you will see preview of it before you commit to it.)
1
Move 1 yields
  X  |     |     
-----|-----|-----
     |     |     
-----|-----|-----
     |     |     

Is this what you want (Y/n)?
y
next command: [n, l, r, s] (with optional /<game>)
? 
```

For convenience, this interface does store the current game state and reuses it by default,
so that you do not have to type very much to interact with it.

However, all of the commands support the full-fledged path syntax that you can use to
override the current state, like so:

```text
next command: [n, l, r, s] (with optional /<game>)
? l/-X-------
list "-X-------" : [(1, "OX-------"), (3, "-XO------"), (4, "-X-O-----"), (5, "-X--O----"), (6, "-X---O---"), (7, "-X----O--"), (8, "-X-----O-"), (9, "-X------O")]
choose a move from list above
(you will see preview of it before you commit to it.)
2
The number 2 is not in the list
Please try again.
list "-X-------" : [(1, "OX-------"), (3, "-XO------"), (4, "-X-O-----"), (5, "-X--O----"), (6, "-X---O---"), (7, "-X----O--"), (8, "-X-----O-"), (9, "-X------O")]
choose a move from list above
(you will see preview of it before you commit to it.)
1
Move 1 yields
  O  |  X  |     
-----|-----|-----
     |     |     
-----|-----|-----
     |     |     

Is this what you want (Y/n)?
y
next command: [n, l, r, s] (with optional /<game>)
? r/-X------O
render "-X------O" :
     |  X  |     
-----|-----|-----
     |     |     
-----|-----|-----
     |     |  O  

next command: [n, l, r, s] (with optional /<game>)
? q
```
