# Universal Backgammon Interface (UBGI) Protocol (Draft 0.01)
## Author: Øystein Schønning-Johansen 2025
Universal Backgammon Interface is a protocol definition to communicate between a UI and a backgammon engine.

This document will describe how a backgammon engine and a user interface (UI) can communicate with each other to separate the concerns of the two processes.

Basically, the engine should be able to do three operations.

  * Find the best action.
    This is - given the position and match info - return the best action. This can be the move to play or the cube action to take.

  * Analyse the position. (Optional)
        Analyse the given position and report the probabilities back to the GUI.

  * Preform a rollout. (Optional)
        Doing rollout controlled by the GUI will give a lot of overhead - it is therefore clear that the rollouts should be conducted by the engine.
        A rollout will typically be time consuming and will need to periodically send results back to the UI. The UI should also be able to break off
        a rollout in progress.

The protocol should also be able to identify it self with with author and version. It should be able to initialize itself and shutdown on command from the UI.
The UI should be able to request a status of the engine.

Options for the engine is an important topic. In future releases of this protocol standard, there will probably be a protocol for the UI to ask
for the engines options. The engine replies with the optinons available (typically search depth, pruning options, cache size etc.) and the UI will then be
able to set optinons based on this info. This is not forgotten, but it will be added to the protocol at a later stage. Meanwhile settings and
options for the engine can be controlled by a settings file at engine startup or hard coded or whatever the the engine developer finds suitable.

The UI is considered the process in charge - the UI will therefore start the engine process and 'pipe' instructions to the engine. The engine is in this
sense "stateless" and has no concept of the game.

## Commands form the UI to the engine:

ubgi -  This is the first command the UI should send to the engine. It tells the engine that the UBGI interface is to be used. After recieving this
                command the engine should run any initialization of the engine. It is important that engine developers do not initialize anything before
                this command is given. The engine should then answer with id name, author and version - and then 'readyok' when engine is successfully
                initialized. If the engine is not initialized correctly, that's a problem of the engine and the engine should log that it's error logs,
                on these occations the engine can say 'sorry' back to the UI, but the UI should not care.
stop -  This command will stop the engine in whatever it is doing. Maybe not so important when finding the best action, but it is important that
                the UI can stop an ongoing analysis or rollout.
quit -  This will ask the engine to cleanup the resources and stop the execution (end the engine process). The pipe will be closed.
<pos>   A position can be sent from the UI to the engine using a fibs rawboard string or a XGID board string. When a string is recieved by
                the engine, it should reply with the best action.

## Commands (reply) from engine to the UI:

id name <string> - The name of the engine.
id author <string> - Then author of the engine
id version <string> - The version of the engine
readyok - A 'readyok' from the eninge to the UI means that the engine is ready to process new commands.
sorry - a command sent back to the UI if something goes wrong or is not supported.

### Typical responses to a position being sent to the engine. Typically the best action.
"move" - A move string that indicates the engines best move (more on the move format below)
take - command to indicate that a offered doubling should be accepted.
drop - command to indicate that a cube should be rejected.
double - command that indicate that the position should be doubeled.
roll - command the that indicate the the position not be doubeled. Hence 'roll' the dice.
<blank> - if there is no legal move in the position given a blank line should be sent.

Any of the responses "move", take, drop, double or roll does not require to be followed by a 'readyok'.

Any command that is the UI sends that the engine cannot interpret should either be ignored or respond with a polite 'sorry'

## Board format
The engine should read XGID stings of boars and/or fibs rawboard string.
The fibs rawboard string is documented here:
http://www.fibs.com/macfibs/help/fibsCommands/pages/rawboard.html
The XGID fromat is documented here:
https://www.extremegammon.com/extremegammon2.pdf (page 146)
Both this formats include a state of dice roll and cube and match and other rules.

## Move format
The move format should be simple to parse for the UI and simple for the engine to write. The moves should therfore consist of up to
eight numbers in up to four pairs of from-to points. Points are numbered from the point of view of the player on move. The bar is
therefore considered the 25 point. The the from-to pairs should be written all out ie. a leaping roll where one checker is moved
twice (or more), say opening with 65 and playing 24/13 should be sent as 24/18 18/13. Repeating moves, typically when rolling
doubles, should *not* be repeated by parenthesis. Like 33: 13/10(2) 8/5(2). This should be written out as 13/10 13/10 8/5 8/5.
Repeated moves with parentheses complicates both generation of the move string and the parsing. There should not be a * to
indicate a hit. This also complicates the move generation and the parsing. Writing out all the partial moves is consistent
and will pick up the hitting part of a move anyway. The from-to pair can be separated by either '-', '/' or ' '. Each pair
(partial move) should be separated by a space. 25 and 'bar' can be used interchangeably, and the same for 'off' and 0 - however
no capital letters.

### Example of good formatted move strings
    8/5 6/5
    8 5 6 5
    8-5 6-5
