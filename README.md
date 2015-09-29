# Matlab
matlab functions that manipulate equity data
%%% cycles a list of listed option STOCK symbols through jump_check
%   and builds a cell array of the output
function[list] = symbol_loop(option_symbols,n)
s = size(option_symbols);
list = {};
    for i = 1:s(1)
        use = char(option_symbols(i));
        pop = jump_check(use, n);      
        list{i} = pop;
    end        
end
_____________________________________________________________________________________________________
%%% finds high of stock for period 1/1/2013 to today's date
% looks n days ahead of the high and calculates the percent increase
% output returns a type CHAR of symbol if the percent increase was >= 200% 
% 'NA' is returned otherwise
function[confirm] = jump_check(stock,n_days)
matrix_data = hist_stock_data('01012013', date, stock);
%return 'NA' if there is no data in the cell
if size(matrix_data) == [0,0]
    confirm = 0;
    return
end
vector = matrix_data.Close;
high = max(vector);
spot = find(vector == high);
s = size(spot);
%returns '>1High' if the high closing price is repeated more than once
if s(1) > 1
    confirm = 0;
    return
end
%returns 'NA' if there isn't n days data before the high close
if spot + n_days > length(vector) 
    confirm = 0;
    return
end
n_days_prev_close = vector(spot + n_days);
jump = high / n_days_prev_close;
if jump >= 2
    confirm = stock;
else
    confirm = 0
end
end  
__________________________________________________________________________________________________________________
function stocks = hist_stock_data(start_date, end_date, varargin)
% HIST_STOCK_DATA     Obtain historical stock data
%   hist_stock_data(X,Y,'Ticker1','Ticker2',...) retrieves historical stock
%   data for the ticker symbols Ticker1, Ticker2, etc... between the dates
%   specified by X and Y.  X and Y are strings in the format ddmmyyyy,
%   where X is the beginning date and Y is the ending date.  The program
%   returns the stock data in a structure giving the Date, Open, High, Low,
%   Close, Volume, and Adjusted Close price adjusted for dividends and
%   splits.
%
%   hist_stock_data(X,Y,'tickers.txt') retrieves historical stock data
%   using the ticker symbols found in the user-defined text file.  Ticker
%   symbols must be separated by line feeds.
%
%   EXAMPLES
%       stocks = hist_stock_data('23012003','15042008','GOOG','C');
%           Returns the structure array 'stocks' that holds historical
%           stock data for Google and CitiBank for dates from January
%           23, 2003 to April 15, 2008.
%
%       stocks = hist_stock_data('12101997','18092001','tickers.txt');
%           Returns the structure arrary 'stocks' which holds historical
%           stock data for the ticker symbols listed in the text file
%           'tickers.txt' for dates from October 12, 1997 to September 18,
%           2001.  The text file must be a column of ticker symbols
%           separated by new lines.
%
%       stocks = hist_stock_data('12101997','18092001','C','frequency','w')
%           Returns historical stock data for Citibank using the date range
%           specified with a frequency of weeks.  Possible values for
%           frequency are d (daily), w (weekly), or m (monthly).  If not
%           specified, the default frequency is daily.
%
%   DATA STRUCTURE
%       INPUT           DATA STRUCTURE      FORMAT
%       X (start date)  ddmmyyyy            String
%       Y (end date)    ddmmyyyy            String
%       Ticker          NA                  String 
%       ticker.txt      NA                  Text file
%
%   OUTPUT FORMAT
%       All data is output in the structure 'stocks'.  Each structure
%       element will contain the ticker name, then vectors consisting of
%       the organized data sorted by date, followed by the Open, High, Low,
%       Close, Volume, then Adjusted Close prices.
%
%   DATA FEED
%       The historical stock data is obtained using Yahoo! Finance website.
%       By using Yahoo! Finance, you agree not to redistribute the
%       information found therein.  Therefore, this program is for personal
%       use only, and any information that you obtain may not be
%       redistributed.
%
%   NOTE
%       This program uses the Matlab command urlread in a very basic form.
%       If the program gives you an error and does not retrieve the stock
%       information, it is most likely because there is a problem with the
%       urlread command.  You may have to tweak the code to let the program
%       connect to the internet and retrieve the data.

% Created by Josiah Renfree
% January 25, 2008

stocks = struct([]);        % initialize data structure

% split up beginning date into day, month, and year.  The month is
% subracted is subtracted by 1 since that is the format that Yahoo! uses
bd = start_date(1:2);       % beginning day
bm = sprintf('%02d',str2double(start_date(3:4))-1); % beginning month
by = start_date(5:8);       % beginning year

% split up ending date into day, month, and year.  The month is subracted
% by 1 since that is the format that Yahoo! uses
ed = end_date(1:2);         % ending day
em = sprintf('%02d',str2double(end_date(3:4))-1);   % ending month
ey = end_date(5:8);         % ending year

% determine if user specified frequency
temp = find(strcmp(varargin,'frequency') == 1); % search for frequency
if isempty(temp)                            % if not given
    freq = 'd';                             % default is daily
else                                        % if user supplies frequency
    freq = varargin{temp+1};                % assign to user input
    varargin(temp:temp+1) = [];             % remove from varargin
end
clear temp

% Determine if user supplied ticker symbols or a text file
if isempty(strfind(varargin{1},'.txt'))     % If individual tickers
    tickers = varargin;                     % obtain ticker symbols
else                                        % If text file supplied
    fid = fopen(varargin{1}, 'r');
    tickers = textscan(fid, '%s'); tickers = tickers{:};
    fclose(fid);
end

h = waitbar(0, 'Please Wait...');           % create waitbar
idx = 1;                                    % idx for current stock data

% Cycle through each ticker symbol and retrieve historical data
for i = 1:length(tickers)
    
    % Update waitbar to display current ticker
    waitbar((i-1)/length(tickers),h,sprintf('%s %s %s%0.2f%s', ...
        'Retrieving stock data for',tickers{i},'(',(i-1)*100/length(tickers),'%)'))
        
    % Download historical data using the Yahoo! Finance website
    [temp, status] = urlread(strcat('http://ichart.finance.yahoo.com/table.csv?s='...
        ,tickers{i},'&a=',bm,'&b=',bd,'&c=',by,'&d=',em,'&e=',ed,'&f=',...
        ey,'&g=',freq,'&ignore=.csv'));
    
    % If data was downloaded successfulle, then proceed to process it.
    % Otherwise, ignore this ticker symbol
    if status
        
        % Parse out the historical data
        data = textscan(temp, '%s%f%f%f%f%f%f', 'delimiter', ',', ...
            'Headerlines', 1);
        
        % Put data into appropriate variables
        [stocks(idx).Date, stocks(idx).Open, stocks(idx).High, ...
            stocks(idx).Low, stocks(idx).Close, stocks(idx).Volume, ...
            stocks(idx).AdjClose] = deal(data{:});

        stocks(idx).Ticker = tickers{i};	% Store ticker symbol
        
        idx = idx + 1;                      % Increment stock index
    end
        
    % update waitbar
    waitbar(i/length(tickers),h)
end
close(h)    % close waitbar

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

function [NOTEAlldirectoriesareupdateddailyusinginformationfromthepreviou,VarName2] = importCBOEfile(filename, startRow, endRow)
%IMPORTFILE Import numeric data from a text file as column vectors.
%   [NOTEALLDIRECTORIESAREUPDATEDDAILYUSINGINFORMATIONFROMTHEPREVIOU,VARNAME2]
%   = IMPORTFILE(FILENAME) Reads data from text file FILENAME for the
%   default selection.
%
%   [NOTEALLDIRECTORIESAREUPDATEDDAILYUSINGINFORMATIONFROMTHEPREVIOU,VARNAME2]
%   = IMPORTFILE(FILENAME, STARTROW, ENDROW) Reads data from rows STARTROW
%   through ENDROW of text file FILENAME.
%
% Example:
%   [NOTEAlldirectoriesareupdateddailyusinginformationfromthepreviou,VarName2]
%   = importfile('cboesymboldir2.csv',3, 3958);
%
%    See also TEXTSCAN.

% Auto-generated by MATLAB on 2015/09/23 21:31:30

%% Initialize variables.
delimiter = ',';
if nargin<=2
    startRow = 3;
    endRow = inf;
end

%% Read columns of data as strings:
% For more information, see the TEXTSCAN documentation.
formatSpec = '%s%s%[^\n\r]';

%% Open the text file.
fileID = fopen(filename,'r');

%% Read columns of data according to format string.
% This call is based on the structure of the file used to generate this
% code. If an error occurs for a different file, try regenerating the code
% from the Import Tool.
dataArray = textscan(fileID, formatSpec, endRow(1)-startRow(1)+1, 'Delimiter', delimiter, 'HeaderLines', startRow(1)-1, 'ReturnOnError', false);
for block=2:length(startRow)
    frewind(fileID);
    dataArrayBlock = textscan(fileID, formatSpec, endRow(block)-startRow(block)+1, 'Delimiter', delimiter, 'HeaderLines', startRow(block)-1, 'ReturnOnError', false);
    for col=1:length(dataArray)
        dataArray{col} = [dataArray{col};dataArrayBlock{col}];
    end
end

%% Close the text file.
fclose(fileID);

%% Convert the contents of columns containing numeric strings to numbers.
% Replace non-numeric strings with NaN.
raw = repmat({''},length(dataArray{1}),length(dataArray)-1);
for col=1:length(dataArray)-1
    raw(1:length(dataArray{col}),col) = dataArray{col};
end
numericData = NaN(size(dataArray{1},1),size(dataArray,2));


%% Split data into numeric and cell columns.
rawNumericColumns = {};
rawCellColumns = raw(:, [1,2]);


%% Replace non-numeric cells with NaN
R = cellfun(@(x) ~isnumeric(x) && ~islogical(x),rawNumericColumns); % Find non-numeric cells
rawNumericColumns(R) = {NaN}; % Replace non-numeric cells

%% Allocate imported array to column variable names
NOTEAlldirectoriesareupdateddailyusinginformationfromthepreviou = rawCellColumns(:, 1);
VarName2 = rawCellColumns(:, 2);


_________________________________________________________________________________________________________________
