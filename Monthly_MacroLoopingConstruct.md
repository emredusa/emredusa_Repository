/*macro looping construct, particularly helpful capturing month-end close, backdated records usually critical in underwriting adjudication but can also be employed as part of portfolio servicing analytics, or simply stacking daily close and creating time-series*/


%macro time(startdate,enddate); /*see below line 50*/

%let startdate = %sysfunc(inputn(&startdate,yymmdd10.));
%let enddate = %sysfunc(inputn(&enddate,yymmdd10.));

/*%let diff=%sysfunc(intck(weekday,&startdate,&enddate)); /*careful this is a weekday, skips weekends!!*/
/*%do i=0 %to &diff;*/

%let diff=%sysfunc(intck(month,&startdate,&enddate));
%do j=0 %to &diff;
/*
%let bdt=%sysfunc(putn(%sysfunc(intnx(weekday,&startdate,&i,b)),yymmddn8.)); 
%let edt=%sysfunc(putn(%sysfunc(intnx(weekday,&startdate,&i,e)),yymmddn8.));
%let Date=%sysfunc(putn(%sysfunc(intnx(weekday,&startdate,&i,b)),yymmddn8.));
%let Date2=%sysfunc(putn(%sysfunc(intnx(weekday,&startdate,&i,b)),yymmdd10.));
%let Year=%sysfunc(putn(%sysfunc(intnx(weekday,&startdate,&i,b)),Year.)); 
%let MonYear=%sysfunc(putn(%sysfunc(intnx(weekday,&startdate,&i,b)),MonYY7.));*/ /*use what you need based on context and data structure*/
%let MonYear2=%sysfunc(putn(%sysfunc(intnx(month,&startdate,&j,b)),MonYY7.));
%let MonthBeg=%sysfunc(putn(%sysfunc(intnx(month,&startdate,&j,b)),yymmdd10.));
%let MonthEnd=%sysfunc(putn(%sysfunc(intnx(month,&startdate,&j,e)),yymmdd10.));

%put /*&bdt &edt &date &date2 &Year &MonYear*/ &MonYear2 &MonthBeg &MonthEnd; 

options sastrace=',,,ds' sastraceloc=saslog nostsuffix mlogic mprint;
proc sql outobs= max stimer;
connect to odbc (user = &username. password=&pwd. DSN='SAS_VM');
create table YouNameIt_&MonYear2 (compress=yes) as
select * from connection to odbc
(SELECT

(1) as Count,
/*your-data*/
DATE_FORMAT(b.closing_date,'%M - %Y') as Closing_MonYr,
Concat(Year(b.closing_date),'-','Q',Quarter(b.closing_date)) as ClosingYrQtr,
/*your-data*/

FROM 

where 

/*order by  desc*/
);
quit;

%end;
%mend time;
%time (2016-01-01,2016-06-30); /*SELECT YOUR DATE RANGE*/
options sastrace=off;

%PUT &SYSDSN.;
%PUT &SYSLAST.;

options sastrace=',,,ds' sastraceloc=saslog nostsuffix mlogic mprint;
%macro combo(lib,lastDSN);
proc sql noprint;

 %IF %SYSFUNC(EXIST(&lastDSN)) = 1 %THEN %DO;
	select distinct memname
		into :dsns separated by '  '
 			from dictionary.tables
 	where libname=%upcase("&lib");
quit;
%end;

	 		%put &sqlobs data sets in &lib are: &dsns;

		%if &sqlobs. = 0 %then %do;
		%put Nothing to do!;
		%goto bottom;
		%end;
	
	data NAME.YOURDATA;
		set &dsns;
	run;		
%bottom: ;
%Mend;
%combo(Work,&SYSLAST.);
options sastrace=off;

PROC Delete data = work._all_; quit;
