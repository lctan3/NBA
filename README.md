NBA
===

NBA Playoff Simulator
/*
 * File: main.cpp
 * --------------
 * NBA Playoff Simulator
 */

#include <iostream>
#include "console.h"
#include "simpio.h"
#include "random.h"
#include "strlib.h"
#include "map.h"
#include "vector.h"

using namespace std;

struct scoreKey{
    int winnerscore;
    int loserscore;
    string gameWinner;
    string loser;
};

struct scoreCardStruct{
    Map<string,scoreKey> scoreCard;
};

void playGame(string team1, string team2, int& team1wins, int& team2wins, int homeStrength, int awayStrength, int games, scoreKey& gameKey);
void sortTeams(Map<string,Vector<int> >& statsMap, Vector<string>& teamVector);
void addTeam(Map<string,Vector<int> >& statsMap, Vector<string>& teamVector, string teamName, int homeStrength, int awayStrength, int wins);
void printSeriesResults(Map<string,scoreKey > scoreCard);
string playSeries(string teamA, string teamB,Map<string,Vector<int> >& statsMap,Map<string,scoreKey >& scoreCard);
void sortFirstRoundTeams(Vector<string>& teamVector, Vector<string>& firstRoundTeams);
void playRound(Vector<string>& candidateTeams, Vector<string>& winningTeams, Vector<scoreCardStruct>& scoreCards, Map<string,Vector<int> >& statsMap);

const int upperLim = 10;
const int lowerLim = -2;
const int controlDiv = 3;
const int corrHome = 12;
const int corrAway = 11;
const int numSeries = 15;
const int numTeams = 8;

int main(){
    getLine("This program simulates the NBA Playoffs for a given year. Press enter to begin: ");
        
    // statsMap has team name as key and Home/Away score as values
    Map<string,Vector<int> > statsMap;
    Vector<string> teamVector;
    Vector<string> firstRoundTeams;
    Vector<string> confSemisTeams;
    Vector<string> confFinalsTeams;
    Vector<string> theFinalsTeams;
    Vector<string> NBAChampion;
    sortTeams(statsMap, teamVector);
    Vector<scoreCardStruct> scoreCards;

    sortFirstRoundTeams(teamVector, firstRoundTeams);
    
    playRound(firstRoundTeams, confSemisTeams, scoreCards, statsMap);
    playRound(confSemisTeams, confFinalsTeams, scoreCards, statsMap);
    playRound(confFinalsTeams, theFinalsTeams, scoreCards, statsMap);
    playRound(theFinalsTeams, NBAChampion, scoreCards, statsMap);
    
    cout << "NBA Champion: " << NBAChampion[0] << endl;
    
    return 0;
}

void playRound(Vector<string>& candidateTeams, Vector<string>& winningTeams, Vector<scoreCardStruct>& scoreCards, Map<string,Vector<int> >& statsMap){
    while(candidateTeams.size() > 0){
        scoreCardStruct scoreCardStr;
        string team1 = candidateTeams[0];
        string team2 = candidateTeams[1];
        candidateTeams.remove(0);
        candidateTeams.remove(0);
        string winner = playSeries(team1, team2, statsMap, scoreCardStr.scoreCard);
        scoreCards.add(scoreCardStr);
        winningTeams.add(winner);
    }
}

void sortFirstRoundTeams(Vector<string>& teamVector, Vector<string>& firstRoundTeams){
    for (int i=1; i<=8; i++){
        switch(i){
            case 1:
                firstRoundTeams.add(teamVector[0]);
                firstRoundTeams.add(teamVector[7]);
                break;
            case 2:
                firstRoundTeams.add(teamVector[3]);
                firstRoundTeams.add(teamVector[4]);
                break;
            case 3:
                firstRoundTeams.add(teamVector[2]);
                firstRoundTeams.add(teamVector[5]);
                break;
            case 4:
                firstRoundTeams.add(teamVector[1]);
                firstRoundTeams.add(teamVector[6]);
                break;
            case 5:
                firstRoundTeams.add(teamVector[8]);
                firstRoundTeams.add(teamVector[15]);
                break;
            case 6:
                firstRoundTeams.add(teamVector[11]);
                firstRoundTeams.add(teamVector[12]);
                break;
            case 7:
                firstRoundTeams.add(teamVector[10]);
                firstRoundTeams.add(teamVector[13]);
                break;
            case 8:
                firstRoundTeams.add(teamVector[9]);
                firstRoundTeams.add(teamVector[14]);
                break;
        }
    }
}


string playSeries(string teamA, string teamB, Map<string,Vector<int> >& statsMap,Map<string,scoreKey >& scoreCard){
    string team1;
    string team2;
    if (statsMap.get(teamA)[2] >= statsMap.get(teamB)[2]) {
        team1 = teamA;
        team2 = teamB;
    } else {
        team1 = teamB;
        team2 = teamA;
    }
    int team1H = statsMap.get(team1)[0];
    int team2H = statsMap.get(team2)[0];
    int team1A = statsMap.get(team1)[1];
    int team2A = statsMap.get(team2)[1];
    int games = 0;
    int team1wins = 0;
    int team2wins = 0;
    cout << endl;
    while(team1wins<4 && team2wins<4){
        games++;
        scoreKey gameKey;
        switch (games){
            case 1: playGame (team1,team2,team1wins,team2wins,team1H+corrHome,team2A+corrAway,games,gameKey);
                break;
            case 2: playGame (team1,team2,team1wins,team2wins,team1H+corrHome,team2A+corrAway,games,gameKey);
                break;
            case 3: playGame (team1,team2,team1wins,team2wins,team1A+corrAway,team2H+corrHome,games,gameKey);
                break;
            case 4: playGame (team1,team2,team1wins,team2wins,team1A+corrAway,team2H+corrHome,games,gameKey);
                break;
            case 5: playGame (team1,team2,team1wins,team2wins,team1A+corrHome,team2H+corrAway,games,gameKey);
                break;
            case 6: playGame (team1,team2,team1wins,team2wins,team1H+corrAway,team2A+corrHome,games,gameKey);
                break;
            case 7: playGame (team1,team2,team1wins,team2wins,team1H+corrHome,team2A+corrAway,games,gameKey);
                break;
        }
        cout << team1 << ": " << team1wins<< endl;
        cout << team2 << ": " << team2wins<< endl;
        string toAdd = "Game ";
        toAdd += integerToString(games);
        scoreCard.put(toAdd,gameKey);
        getLine("");
    }
    string teamWinner;
    if (team1wins>team2wins){
        cout << team1 << " wins series " << team1wins << "-" << team2wins<< endl;
        teamWinner = team1;
    }
    else {
        cout << team2 << " wins series " << team2wins << "-" << team1wins<< endl;
        teamWinner = team2;
    }
    printSeriesResults(scoreCard);
    return teamWinner;
}

// Scale: 1-7 (7 = Excellent, 1 = Poor)
void sortTeams(Map<string,Vector<int> >& statsMap, Vector<string>& teamVector){
    for (int i=1; i<=16; i++){
        switch(i){
            case 1: addTeam(statsMap,teamVector, "SAS(1)",7,6,65);
                break;
            case 2: addTeam(statsMap,teamVector, "OKC(2)",7,6,61);
                break;
            case 3: addTeam(statsMap,teamVector,"LAC(3)",6,6,58);
                break;
            case 4: addTeam(statsMap,teamVector,"GSW(4)",6,6,57);
                break;
            case 5: addTeam(statsMap,teamVector,"HOU(5)",6,6,55);
                break;
            case 6: addTeam(statsMap,teamVector,"DAL(6)",5,5,51);
                break;
            case 7: addTeam(statsMap,teamVector,"POR(7)",5,5,49);
                break;
            case 8: addTeam(statsMap,teamVector,"MEM(8)",5,5,43);
                break;
            case 9: addTeam(statsMap,teamVector,"MIA(1)",7,6,67);
                break;
            case 10: addTeam(statsMap,teamVector,"CHI(2)",7,6,62);
                break;
            case 11: addTeam(statsMap,teamVector,"IND(3)",7,6,56);
                break;
            case 12: addTeam(statsMap,teamVector,"BKN(4)",6,6,50);
                break;
            case 13: addTeam(statsMap,teamVector,"WAS(5)",6,6,48);
                break;
            case 14: addTeam(statsMap,teamVector,"CLE(6)",5,5,44);
                break;
            case 15: addTeam(statsMap,teamVector,"CHA(7)",5,5,41);
                break;
            case 16: addTeam(statsMap,teamVector,"TOR(8)",5,5,37);
                break;
        }
    }
}

void addTeam(Map<string,Vector<int> >& statsMap, Vector<string>& teamVector, string teamName, int homeStrength, int awayStrength, int wins){
    Vector<int> stats;
    //0th entry = homestrength
    //1st entry = awaystrength
    //2nd entry = team seeding
    stats.add(homeStrength);
    stats.add(awayStrength);
    stats.add(wins);
    statsMap.put(teamName,stats);
    teamVector.add(teamName);
}

void playGame(string team1, string team2, int& team1wins, int& team2wins, int team1Strength, int team2Strength, int games, scoreKey& gameKey){
    cout << team1 << " Versus " << team2 << " Game " << games << endl;
    getLine();
    int team1score = 0;
    int team2score = 0;
    int otcounter = 0;
    for (int i=1; i<=4; i++){
        if (abs(team1score-team2score)<12){
            team1score+= team1Strength+randomInteger(lowerLim,upperLim);
            team2score+= team2Strength+randomInteger(lowerLim,upperLim);
        } else {
            int corrFactor = abs(team2score-team1score)/controlDiv;
            if (team1score<team2score){
                team1score+= team1Strength+randomInteger(lowerLim+corrFactor,upperLim+corrFactor);
                team2score+= team2Strength+randomInteger(lowerLim,upperLim);
            } else {
                team1score+= team1Strength+randomInteger(lowerLim,upperLim);
                team2score+= team2Strength+randomInteger(lowerLim+corrFactor,upperLim+corrFactor);
            }
        }
        team1score += randomInteger(-2, 5);
        team2score += randomInteger(-2, 5);
        
        cout << "Q" << i << endl;
        cout << team1 << ":" << team1score << endl;
        cout << team2 << ":" << team2score << endl;
        //Comment out for fast mode
        getLine();
       
    }

    while (team1score==team2score) {
        otcounter++;
        team1score+=team1Strength/2+randomInteger(lowerLim,upperLim/5);
        team2score+=team2Strength/2+randomInteger(lowerLim,upperLim/5);
        cout << "OT" << otcounter << endl;
        cout << team1 << ":" << team1score << endl;
        cout << team2 << ":" << team2score << endl;
        getLine();
    }
    cout << team1 << ": " << team1score << " - " << team2 << ": " << team2score << endl;
    if (team1score>team2score){
        cout << team1 << " wins " << "Game " << games << endl;
        team1wins++;
        gameKey.gameWinner = team1;
        gameKey.winnerscore = team1score;
        gameKey.loser = team2;
        gameKey.loserscore = team2score;
    } else{
        cout << team2 << " wins " << "Game " << games << endl;
        team2wins++;
        gameKey.gameWinner = team2;
        gameKey.winnerscore = team2score;
        gameKey.loser = team1;
        gameKey.loserscore = team1score;
    }
}

// Scorecard has Game number and Winner/Loser scores/names (added individually for each game)
void printSeriesResults(Map<string,scoreKey > scoreCard){
    foreach(string game in scoreCard){
        scoreKey toPrint = scoreCard.get(game);
        cout << game << ": " << toPrint.gameWinner << " " << toPrint.winnerscore << " - " << toPrint.loser << " " << toPrint.loserscore << endl;
    }
}
