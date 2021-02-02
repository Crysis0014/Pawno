/*
   Аренда авто [Rent a car] v 0.1
   |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
   || Автор: Spirit                                                           ||
   || Благодарность за помощь: OFFREAL, OkStyle                               ||
   |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
*/
#include <a_samp>
//==============================================================================
#define R_F                                                                  943//Ид диалога
#define COLOR_WHITE                                                   0xFFFFFFAA//Цвет (белый)
#define MAX_RENT_VEH                                                          30//Максимум авто на аренду
//==============================================================================
enum RInfo
{
    Rcarid,
    Rprice,
    ROwned
};
new RentCar[MAX_RENT_VEH][RInfo];
new IsRentableVehicle[MAX_VEHICLES];
new OwnedName[MAX_PLAYER_NAME];
new PlayerUseRentCar[MAX_PLAYERS];


public OnFilterScriptInit()
{
    print("==========================================");
	print("[!]Rent Cars by Spirit Sucsessfuly loaded!");
	print("==========================================");
	SetTimer("CountRentedVehicles", 1000, 0);
/*    [Авто для аренды. Добавляйте свои ориентируясь на написанный ниже пример.] */
/*Параметры добавления: -AddRentVehicle(ид, модель, X, Y, Z, Rot, Цвет1, Цвет2, цена);*/
//==============================================================================
	AddRentVehicle(1,462,1800.0,-1934.0,13.07,360.0,1,1,150); //Пример 1
    AddRentVehicle(2,462,1796.0,-1934.0,13.07,360.0,2,2,150); //Пример 2
   	AddRentVehicle(3,462,1792.0,-1934.0,13.07,360.0,3,3,150); //Пример 3
   	AddRentVehicle(4,462,1788.0,-1934.0,13.07,360.0,4,4,150); //Пример 3
   	AddRentVehicle(5,462,1784.0,-1934.0,13.07,360.0,5,5,150); //Пример 3
//==============================================================================
	return 1;
}
public OnFilterScriptExit()
{
    print("[!]Rent Cars by Spirit Unloaded!");
	return 1;
}
public OnVehicleDeath(vehicleid, killerid)
{
    
	return 1;
}
public OnPlayerStateChange(playerid, newstate, oldstate)
{
    new rcs[265];
	if(newstate == 2)
    {
		if(RentCar[GetPlayerVehicleID(playerid)][ROwned] != 1 && IsRentableVehicle[GetPlayerVehicleID(playerid)])// == GetPlayerName(playerid, OwnedName, sizeof(OwnedName)))
		{
			if(PlayerUseRentCar[playerid] == 1)
			{
			    SendClientMessage(playerid,-1,"|Вы не можете арендовать еще один скутер.");
			    SendClientMessage(playerid,-1,"|У вас уже есть арендованый скутер.");
				RemovePlayerFromVehicle(playerid);
				TogglePlayerControllable(playerid,1);
				return 1;
			}
			format(rcs, sizeof(rcs), "{ffffff}Здравствуйте! Этот скутер сдаётся в аренду!\n Порядковый номер - [{00ceff}%d{ffffff}]\n Цена аренды [{5da130}$%d{ffffff}]\n\nВы хотите арендовать данный скутер?",RentCar[GetPlayerVehicleID(playerid)][Rcarid],RentCar[GetPlayerVehicleID(playerid)][Rprice]);
			ShowPlayerDialog(playerid,R_F,DIALOG_STYLE_MSGBOX,"Аренда",rcs,"Арендовать","Отмена");
		}
		if(RentCar[GetPlayerVehicleID(playerid)][ROwned] != 0 && IsRentableVehicle[GetPlayerVehicleID(playerid)])
		{
		    if(RentCar[GetPlayerVehicleID(playerid)][ROwned] == GetPlayerName(playerid, OwnedName, sizeof(OwnedName)))
		    {
                 GetPlayerName(playerid, OwnedName, sizeof(OwnedName));
                 format(rcs, sizeof(rcs), "Этот скутер арендован на {ff8700}%s{ffffff}.",OwnedName);
                 SendClientMessage(playerid,-1,rcs);
			}
			if(RentCar[GetPlayerVehicleID(playerid)][ROwned] != GetPlayerName(playerid, OwnedName, sizeof(OwnedName)))
			{
                 SendClientMessage(playerid,-1,"Скутер уже арендован другим игроком.");
                 RemovePlayerFromVehicle(playerid);
		         TogglePlayerControllable(playerid,1);
			}
	    }
	}
    return 1;
}


public OnDialogResponse(playerid, dialogid, response, listitem, inputtext[])
{
    if(dialogid == R_F)
     {
         if(response)
         {
	         new carprice = RentCar[GetPlayerVehicleID(playerid)][Rprice],string[80];
	         if(GetPlayerMoney(playerid) < carprice)
	         {
                 RemovePlayerFromVehicle(playerid);
                 TogglePlayerControllable(playerid,1);
                 SendClientMessage(playerid,COLOR_WHITE,"У вас недостаточно средств для аренды скутера.");
                 return 1;
	         }
             format(string, sizeof(string), "Вы арендовали это скутер средство за  - {5da130}$%d{ffffff}.", carprice);
             SendClientMessage(playerid,COLOR_WHITE,string);
             TogglePlayerControllable(playerid,1);
             GivePlayerMoney(playerid,-carprice);
             PlayerUseRentCar[playerid] = 1;
		     RentCar[GetPlayerVehicleID(playerid)][ROwned] = GetPlayerName(playerid, OwnedName, sizeof(OwnedName));
		     SetVehicleNumberPlate(GetPlayerVehicleID(playerid),"RENTED");
        }
            else
        {
            RemovePlayerFromVehicle(playerid);
		    TogglePlayerControllable(playerid,1);
		    PlayerUseRentCar[playerid] = 0;
        }
   }
    return 1;
}

stock AddRentVehicle(id ,model, Float:X, Float:Y, Float:Z, Float:Angle, color1, color2, price)
{
    new newvid;
    newvid = AddStaticVehicle(model, X, Y, Z, Angle, color1, color2);
    RentCar[newvid][Rprice] = price;
    RentCar[newvid][Rcarid] = id;
    RentCar[newvid][ROwned] = 0;
    IsRentableVehicle[newvid] = 1;
    SetVehicleNumberPlate(newvid,"RENT");
}


TotalVehicles()
{
	new vid;
	vid = CreateVehicle(411, 0, 0, 0, 0, -1, -1, 10);
	DestroyVehicle(vid);
	vid--;
	return vid;
}
forward CountRentedVehicles();
public CountRentedVehicles()
{
	new count;
	for(new R=1; R<TotalVehicles(); R++)
	{
	    if(IsRentableVehicle[R] == 1)
	    {
	        count++;
		}
	}
}
