# Risk Analyis in Roulette Game with Random Outcomes
Code Discription of This Game




function ExtendedRouletteGameApp
   % Create the main UI window
fig = uifigure('Name', 'Risk Analysis Roulette Game with Random Outcomes', 'Position', [100, 100, 800, 600]);

% Title Label
uilabel(fig, 'Position', [200, 550, 400, 30], 'Text', 'Risk Analysis Roulette Game with Random Outcomes', ...
    'FontSize', 16, 'FontWeight', 'bold', 'HorizontalAlignment', 'center');


    % Bet type dropdown
    uilabel(fig, 'Position', [20, 500, 100, 30], 'Text', 'Bet Type:');
    betType = uidropdown(fig, 'Position', [120, 500, 200, 30], ...
        'Items', {'Straight', 'Split', 'Street', 'Line', ...
                  'Dozen', 'Column', 'Red/Black', 'Even/Odd', 'High/Low'});

    % Bet details input
    uilabel(fig, 'Position', [20, 460, 100, 30], 'Text', 'Bet Details:');
    betDetails = uieditfield(fig, 'text', 'Position', [120, 460, 200, 30]);

    % Bet amount input
    uilabel(fig, 'Position', [20, 420, 100, 30], 'Text', 'Bet Amount:');
    betAmount = uieditfield(fig, 'numeric', 'Position', [120, 420, 200, 30]);

    % Spin button
    uibutton(fig, 'Position', [20, 380, 100, 30], 'Text', 'Spin', ...
        'ButtonPushedFcn', @(btn, event) spinRoulette());

    % Restart button
    uibutton(fig, 'Position', [150, 380, 100, 30], 'Text', 'Restart', ...
        'ButtonPushedFcn', @(btn, event) restartGame());

    % Simulation rounds input
    uilabel(fig, 'Position', [20, 340, 150, 30], 'Text', 'Simulation Rounds:');
    numRounds = uieditfield(fig, 'numeric', 'Position', [170, 340, 150, 30], 'Value', 1);

    % Result label
    resultLabel = uilabel(fig, 'Position', [20, 300, 300, 30], ...
        'Text', 'Result: ', 'FontSize', 12);

    % Spin Result Color Label
    spinColorLabel = uilabel(fig, 'Position', [20, 270, 300, 30], ...
        'Text', 'Spin Color: ', 'FontSize', 12);

    % Risk Analysis Area
    ax = uiaxes(fig, 'Position', [350, 180, 400, 350]);
    title(ax, 'Profit Distribution');
    xlabel(ax, 'Profit ($)');
    ylabel(ax, 'Frequency');

    % Risk Analysis Labels
    uilabel(fig, 'Position', [20, 260, 120, 30], 'Text', 'Expected Value:');
    expectedValueLabel = uilabel(fig, 'Position', [150, 260, 100, 30], 'Text', '0');

    uilabel(fig, 'Position', [20, 230, 120, 30], 'Text', 'Variance:');
    varianceLabel = uilabel(fig, 'Position', [150, 230, 100, 30], 'Text', '0');

    uilabel(fig, 'Position', [20, 200, 120, 30], 'Text', 'Std Deviation:');
    stdDevLabel = uilabel(fig, 'Position', [150, 200, 100, 30], 'Text', '0');

    % Store profits for risk analysis
    profits = [];

    % Callback for spinning the roulette
    function spinRoulette()
        % Validate inputs before starting the rounds
        if isempty(betAmount.Value) || betAmount.Value <= 0
            uialert(fig, 'Enter a valid bet amount.', 'Input Error');
            return;
        end
        if isempty(numRounds.Value) || numRounds.Value <= 0
            uialert(fig, 'Enter a valid number of rounds.', 'Input Error');
            return;
        end
        if isempty(betDetails.Value) || ~ischar(betDetails.Value)
            uialert(fig, 'Enter valid bet details.', 'Input Error');
            return;
        end

        % Roulette setup
        slots = [0, '00', 1:36]; % Roulette slots with "00"
        rounds = numRounds.Value; % Number of spins
        totalProfit = 0; % Total profit for multiple spins

        % Validate bet details once, before any rounds
        validBet = true;
        try
            calculatePayout(slots(randi(length(slots))), betType.Value, betDetails.Value, betAmount.Value); % Dummy call to validate
        catch ME
            uialert(fig, ME.message, 'Input Error');
            validBet = false;
        end

        if ~validBet
            return; % Don't proceed if there's an error in the bet
        end

        profits = []; % Initialize profits array
        for i = 1:rounds
            spinResult = slots(randi(length(slots))); % Random spin result
            [payout, spinColor] = calculatePayout(spinResult, betType.Value, betDetails.Value, betAmount.Value);
            profit = payout - betAmount.Value; % Profit = Payout - Bet Amount
            profits = [profits, profit]; % Store for analysis
            totalProfit = totalProfit + profit;
            
            % Update Spin Color
            spinColorLabel.Text = sprintf('Spin Color: %s', spinColor);
        end

        % Update Results
        resultLabel.Text = sprintf('Result: Total Profit: $%.2f over %d spins', totalProfit, rounds);

        % Update Risk Analysis Metrics
        expectedValueLabel.Text = sprintf('%.2f', mean(profits));
        varianceLabel.Text = sprintf('%.2f', var(profits));
        stdDevLabel.Text = sprintf('%.2f', std(profits));

        % Update Histogram
        cla(ax);
        histogram(ax, profits, 'Normalization', 'pdf');
    end

    % Restart the game and reset all data
    function restartGame()
        % Clear profits
        profits = [];
        
        % Reset input fields
        betType.Value = 'Straight';
        betDetails.Value = '';
        betAmount.Value = 0;
        numRounds.Value = 1;

        % Reset results
        resultLabel.Text = 'Result: ';
        expectedValueLabel.Text = '0';
        varianceLabel.Text = '0';
        stdDevLabel.Text = '0';

        % Reset spin color
        spinColorLabel.Text = 'Spin Color: ';

        % Clear the histogram
        cla(ax);
        title(ax, 'Profit Distribution');
    end

    % Calculate payout based on bet type and spin result
    function [payout, spinColor] = calculatePayout(spinResult, betType, betDetails, betAmount)
        payout = 0; % Default no payout
        spinColor = 'Green'; % Default spin color for "0" and "00"

        % Determine spin color
        if isequal(spinResult, 0) || isequal(spinResult, '00')
            spinColor = 'Green';
        elseif ismember(spinResult, [1, 3, 5, 7, 9, 12, 14, 16, 18, 19, 21, 23, 25, 27, 30, 32, 34, 36])
            spinColor = 'Red';
        else
            spinColor = 'Black';
        end

        try
            switch betType
                case 'Straight'
                    number = str2double(betDetails);
                    if isnan(number) || number < 0 || number > 36
                        error('For a Straight bet, enter a valid number between 0 and 36.');
                    end
                    if number == spinResult
                        payout = betAmount * 35;
                    end
   case 'Split'
        splitNumbers = str2num(betDetails); %#ok<ST2NM>
        if isempty(splitNumbers) || numel(splitNumbers) ~= 2 || ...
           any(splitNumbers < 0 | splitNumbers > 36) || ...
          abs(splitNumbers(1) - splitNumbers(2)) ~= 1 && abs(splitNumbers(1) - splitNumbers(2)) ~= 3 % check adjacency
            error('For a Split bet, enter two valid adjacent numbers between 0 and 36.');
        end
        if ismember(spinResult, splitNumbers)
            payout = betAmount * 17;
        end
        
  case 'Street'
        streetNumber = str2double(betDetails); % Get the row number entered
        if isnan(streetNumber) || streetNumber < 1 || streetNumber > 12
            error('For a Street bet, enter a valid row number between 1 and 12.');
        end
        
        % Define the rows, each row contains three numbers
        rows = {[1 2 3], [4 5 6], [7 8 9], [10 11 12], [13 14 15], [16 17 18], ...
                [19 20 21], [22 23 24], [25 26 27], [28 29 30], [31 32 33], [34 35 36]};
        
        % Get the selected row based on the entered number
        selectedRow = rows{streetNumber};
        
        % Check if the spin result is within the selected row
        if ismember(spinResult, selectedRow)
            payout = betAmount * 11;
        end
        
    case 'Line'
        lineNumber = str2double(betDetails); % Get the row number entered
        if isnan(lineNumber) || lineNumber < 1 || lineNumber > 11
            error('For a Line bet, enter a valid Consecutive row numbers between 1 and 11.');
        end
        
        % Define the rows, each consecutive two rows contains six numbers
        rows = {[1 2 3 4 5 6], [4 5 6 7 8 9], [7 8 9 10 11 12], [10 11 12 13 14 15] ...
            , [13 14 15 16 17 18], [16 17 18 19 29 21],[19 20 21 22 23 24] ...
            , [22 23 24 25 26 27], [25 26 27 28 29 30],[28 29 30 31 32 33] ...
            , [31 32 33 34 35 36]};
        
        % Get the selected row based on the entered number
        selectedRow = rows{lineNumber};
        
        % Check if the spin result is within the selected row
        if ismember(spinResult, selectedRow)
            payout = betAmount * 5;
        end
        
     case 'Dozen'
                    dozen = str2double(betDetails);
                    if isnan(dozen) || dozen < 1 || dozen > 3
                        error('For a Dozen bet, enter a valid dozen (1, 2, or 3).');
                    end
                    if (dozen == 1 && spinResult >= 1 && spinResult <= 12) || ...
                       (dozen == 2 && spinResult >= 13 && spinResult <= 24) || ...
                       (dozen == 3 && spinResult >= 25 && spinResult <= 36)
                        payout = betAmount * 2;
                    end
                case 'Column'
                    column = str2double(betDetails);
                    if isnan(column) || column < 1 || column > 3
                        error('For a Column bet, enter a valid column (1, 2, or 3).');
                    end
                    if (column == 1 && mod(spinResult - 1, 3) == 0) || ...
                       (column == 2 && mod(spinResult - 2, 3) == 0) || ...
                       (column == 3 && mod(spinResult - 3, 3) == 0)
                        payout = betAmount * 2;
                    end
                case 'Red/Black'
                    if ~ismember(betDetails, {'Red', 'Black'})
                        error('For a Red/Black bet, enter either "Red" or "Black".');
                    end
                    redNumbers = [1, 3, 5, 7, 9, 12, 14, 16, 18, 19, 21, 23, 25, 27, 30, 32, 34, 36];
                    if (strcmp(betDetails, 'Red') && ismember(spinResult, redNumbers)) || ...
                       (strcmp(betDetails, 'Black') && ~ismember(spinResult, redNumbers))
                        payout = betAmount * 2;
                    end
                case 'Even/Odd'
                    if ~ismember(betDetails, {'Even', 'Odd'})
                        error('For an Even/Odd bet, enter either "Even" or "Odd".');
                    end
                    if mod(spinResult, 2) == 0 && strcmp(betDetails, 'Even') || ...
                       mod(spinResult, 2) == 1 && strcmp(betDetails, 'Odd')
                        payout = betAmount * 2;
                    end
                case 'High/Low'
                    if ~ismember(betDetails, {'High', 'Low'})
                        error('For a High/Low bet, enter either "High" or "Low".');
                    end
                    if (strcmp(betDetails, 'Low') && spinResult >= 1 && spinResult <= 18) || ...
                       (strcmp(betDetails, 'High') && spinResult >= 19 && spinResult <= 36)
                        payout = betAmount * 2;
                    end
                otherwise
                    error('Unknown bet type.');
            end
        catch ME
            rethrow(ME);
        end
    end
end

