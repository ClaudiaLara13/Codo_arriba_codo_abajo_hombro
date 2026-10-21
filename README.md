# Codo_arriba_codo_abajo_hombro
Codo arriba codo abajo hombro

function gui_cinematica_inversa()
    % --- Ventana Principal ---
    fig = figure('Name', 'Cinemática Inversa 2R - Plano Rotado 90° en Z', ...
                 'NumberTitle', 'off', 'Position', [100, 100, 900, 550]);

    % --- Panel de Control ---
    pnl = uipanel('Title', ' Parámetros de Entrada ', 'Position', [0.02, 0.05, 0.33, 0.9]);

    % Selección de Configuración
    uicontrol(pnl, 'Style', 'text', 'String', 'Configuración del Codo:', ...
              'Position', [10, 470, 250, 18], 'HorizontalAlignment', 'left');
    pop_codo = uicontrol(pnl, 'Style', 'popupmenu', ...
                         'String', {'-- Indefinido --', 'Codo Arriba (Izquierda)', 'Codo Abajo (Derecha)'}, ...
                         'Value', 1, ...
                         'Position', [10, 445, 250, 25], 'Callback', @calcular_y_graficar);

    % --- Coordenadas de la Base (Base x0, Base y0) ---
    uicontrol(pnl, 'Style', 'text', 'String', 'Base x0 (Horizontal Global):', ...
              'Position', [10, 410, 130, 18], 'HorizontalAlignment', 'left');
    txt_xbase = uicontrol(pnl, 'Style', 'edit', 'String', '5.0', ...
                          'Position', [10, 390, 110, 22], 'Callback', @calcular_y_graficar);

    uicontrol(pnl, 'Style', 'text', 'String', 'Base y0 (Vertical Global):', ...
              'Position', [140, 410, 130, 18], 'HorizontalAlignment', 'left');
    txt_ybase = uicontrol(pnl, 'Style', 'edit', 'String', '8.0', ...
                          'Position', [140, 390, 110, 22], 'Callback', @calcular_y_graficar);

    % --- Coordenadas Locales del Codo (Se miden desde la Base) ---
    uicontrol(pnl, 'Style', 'text', 'String', 'Coordenada Local X (Hacia Abajo):', ...
              'Position', [10, 355, 250, 18], 'HorizontalAlignment', 'left');
    txt_xf = uicontrol(pnl, 'Style', 'edit', 'String', '2.0', ...
                       'Position', [10, 335, 250, 22], 'Callback', @calcular_y_graficar);

    uicontrol(pnl, 'Style', 'text', 'String', 'Coordenada Local Y (A la Derecha):', ...
              'Position', [10, 300, 250, 18], 'HorizontalAlignment', 'left');
    txt_yf = uicontrol(pnl, 'Style', 'edit', 'String', '2.0', ...
                       'Position', [10, 280, 250, 22], 'Callback', @calcular_y_graficar);

    % --- Longitudes de los eslabones l1 y l2 ---
    uicontrol(pnl, 'Style', 'text', 'String', 'Longitud l1:', ...
              'Position', [10, 245, 110, 18], 'HorizontalAlignment', 'left');
    txt_l1 = uicontrol(pnl, 'Style', 'edit', 'String', '4.0', ...
                       'Position', [10, 225, 110, 22], 'Callback', @calcular_y_graficar);

    uicontrol(pnl, 'Style', 'text', 'String', 'Longitud l2:', ...
              'Position', [140, 245, 110, 18], 'HorizontalAlignment', 'left');
    txt_l2 = uicontrol(pnl, 'Style', 'edit', 'String', '2.0', ...
                       'Position', [140, 225, 110, 22], 'Callback', @calcular_y_graficar);

    % Panel de Resultados
    txt_resultado = uicontrol(pnl, 'Style', 'text', 'String', '', ...
                              'Position', [10, 10, 250, 190], ...
                              'HorizontalAlignment', 'left', 'FontSize', 9, 'BackgroundColor', [0.94 0.94 0.94]);

    % --- Ejes Gráficos ---
    ax = axes('Parent', fig, 'Position', [0.42, 0.12, 0.53, 0.8]);
    grid(ax, 'on'); axis(ax, 'equal');
    xlabel(ax, 'X Global'); ylabel(ax, 'Y Global');
    title(ax, 'Plano Rotado 90° en Z en la Base');

    % Calcular al abrir
    calcular_y_graficar();

    % --- Función Callback ---
    function calcular_y_graficar(~, ~)
        % 1. Lectura de variables
        X0  = str2double(get(txt_xbase, 'String'));
        Y0  = str2double(get(txt_ybase, 'String'));
        x_loc = str2double(get(txt_xf, 'String')); % X local (hacia abajo)
        y_loc = str2double(get(txt_yf, 'String')); % Y local (a la derecha)
        l1  = str2double(get(txt_l1, 'String'));
        l2  = str2double(get(txt_l2, 'String'));
        opcion_codo = get(pop_codo, 'Value'); % 1: Indefinido, 2: Codo Arriba, 3: Codo Abajo

        % Estado inicial Indefinido
        if opcion_codo == 1
            res_str = sprintf([' Configuración: Indefinida\n\n' ...
                               ' Base (x0, y0): (%.2f, %.2f)\n' ...
                               ' sigma = Indefinido\n' ...
                               ' beta  = Indefinido\n' ...
                               ' gamma = Indefinido\n\n' ...
                               ' theta1 = Indefinido\n' ...
                               ' theta2 = Indefinido'], X0, Y0);
            set(txt_resultado, 'String', res_str);
            cla(ax); 
            hold(ax, 'on');
            % Dibujar solo los ejes principales
            plot(ax, [-2, 12], [0, 0], '-k', 'LineWidth', 1.2, 'HandleVisibility', 'off');
            plot(ax, [0, 0], [-2, 12], '-k', 'LineWidth', 1.2, 'HandleVisibility', 'off');
            grid(ax, 'on'); axis(ax, 'equal'); xlim(ax, [-2, 12]); ylim(ax, [-2, 12]);
            hold(ax, 'off');
            return;
        end

        if isnan(X0) || isnan(Y0) || isnan(x_loc) || isnan(y_loc) || isnan(l1) || isnan(l2)
            errordlg('Por favor, ingresa únicamente valores numéricos válidos.', 'Error de Entrada');
            return;
        end

        % 2. Geometría Pura sobre el plano local rotado
        r_cuadrado = x_loc^2 + y_loc^2;
        r = sqrt(r_cuadrado);

        % Validaciones Geométricas
        if l1 <= 0 || l2 <= 0
            errordlg('Las longitudes l1 y l2 deben ser mayores a cero.', 'Eslabones Inválidos');
            set(txt_resultado, 'String', ' Error: Longitudes igual a cero.');
            cla(ax); grid(ax, 'on');
            return;
        elseif r > (l1 + l2)
            msg = sprintf('¡Punto fuera de alcance!\n\nLa distancia r = %.2f supera el alcance l1 + l2 = %.2f.', r, l1 + l2);
            errordlg(msg, 'Fuera de Alcance');
            set(txt_resultado, 'String', ' Error: Punto fuera de alcance.');
            cla(ax); grid(ax, 'on');
            return;
        elseif r < abs(l1 - l2)
            msg = sprintf('¡Punto demasiado cercano!\n\nLa distancia r = %.2f es menor a |l1 - l2| = %.2f.', r, abs(l1 - l2));
            errordlg(msg, 'Zona Muerta');
            set(txt_resultado, 'String', ' Error: Punto demasiado cercano.');
            cla(ax); grid(ax, 'on');
            return;
        elseif r == 0
            errordlg('El objetivo del codo no puede ser el (0,0) local.', 'Indeterminación');
            set(txt_resultado, 'String', ' Error: Punto en el origen local.');
            cla(ax); grid(ax, 'on');
            return;
        end

        % Ángulos internos (Fórmulas exactas del pizarrón)
        sigma = atan2d(y_loc, x_loc); % Ángulo medido desde el eje X local (hacia abajo)

        cos_gamma = (l1^2 + l2^2 - r_cuadrado) / (2 * l1 * l2);
        cos_gamma = max(-1, min(1, cos_gamma));
        gamma = acosd(cos_gamma);

        cos_beta = (l1^2 + r_cuadrado - l2^2) / (2 * l1 * r);
        cos_beta = max(-1, min(1, cos_beta));
        beta = acosd(cos_beta);

        % Configuración del Codo
        if opcion_codo == 2
            % Codo Arriba (Izquierda en la referencia rotada)[cite: 3]
            theta1 = beta + sigma;          % theta1 = beta + sigma[cite: 3]
            theta2 = 180 - gamma;           % theta2 = 180° - gamma[cite: 3]
            theta2_abs = theta1 - 180 + gamma;
            nombre_config = 'Codo Arriba';
        elseif opcion_codo == 3
            % Codo Abajo (Derecha en la referencia rotada)[cite: 3]
            theta1 = sigma - beta;          % theta1 = sigma - beta[cite: 3]
            theta2 = 180 - gamma;           % theta2 = 180° - gamma[cite: 3]
            theta2_abs = theta1 + 180 - gamma;
            nombre_config = 'Codo Abajo';
        end

        % 3. Mapeo del sistema local rotado 90° al sistema global de la pantalla:
        % Eje X local (hacia abajo)  --> Resta a Y0 global[cite: 4]
        % Eje Y local (a la derecha) --> Suma a X0 global[cite: 4]
        x1_local = l1 * cosd(theta1);
        y1_local = l1 * sind(theta1);

        x2_local = x1_local + l2 * cosd(theta2_abs);
        y2_local = y1_local + l2 * sind(theta2_abs);

        % Transformación global para dibujar
        P0 = [X0, Y0];
        P1 = [X0 + y1_local, Y0 - x1_local];
        P2 = [X0 + y2_local, Y0 - x2_local];

        % 4. Desplegar resultados
        res_str = sprintf([' Configuración: %s\n\n' ...
                           ' Base (x0, y0): (%.2f, %.2f)\n' ...
                           ' Objetivo Local: (x=%.2f, y=%.2f)\n\n' ...
                           ' sigma = %.2f°\n' ...
                           ' beta  = %.2f°\n' ...
                           ' gamma = %.2f°\n\n' ...
                           ' theta1 = %.2f°\n' ...
                           ' theta2 = %.2f°'], ...
                          nombre_config, X0, Y0, x_loc, y_loc, sigma, beta, gamma, theta1, theta2);
        set(txt_resultado, 'String', res_str);

        % 5. Renderizado gráfico
        cla(ax);
        hold(ax, 'on');

        % --- LÍNEAS DE REFERENCIA DE LA BASE (Como en tu dibujo) ---[cite: 4]
        plot(ax, [0, X0], [Y0, Y0], '--k', 'LineWidth', 1, 'HandleVisibility', 'off'); % Línea Base x0[cite: 4]
        plot(ax, [X0, X0], [0, Y0], '--k', 'LineWidth', 1, 'HandleVisibility', 'off'); % Línea Base y0[cite: 4]

        % --- EJES CARTESIANOS GLOBALES PRINCIPALES ---
        plot(ax, [-5, X0 + l1 + l2 + 5], [0, 0], '-k', 'LineWidth', 1.5, 'HandleVisibility', 'off'); % Eje X Global
        plot(ax, [0, 0], [-5, Y0 + 5], '-k', 'LineWidth', 1.5, 'HandleVisibility', 'off');          % Eje Y Global

        % --- EJES LOCALES ROTADOS EN LA BASE ---[cite: 4]
        plot(ax, [X0, X0 + l1 + l2 + 1], [Y0, Y0], '-.k', 'LineWidth', 1, 'HandleVisibility', 'off'); % Eje Y local (horizontal)[cite: 4]
        plot(ax, [X0, X0], [Y0, Y0 - (l1 + l2 + 1)], '-.k', 'LineWidth', 1, 'HandleVisibility', 'off'); % Eje X local (vertical descendente)[cite: 4]
        text(ax, X0 + l1 + l2 + 1.2, Y0, 'y_{local}', 'FontSize', 10, 'FontWeight', 'bold');
        text(ax, X0, Y0 - (l1 + l2 + 1.2), 'x_{local}', 'FontSize', 10, 'FontWeight', 'bold');

        % Línea r local
        plot(ax, [P0(1), P2(1)], [P0(2), P2(2)], '--k', 'LineWidth', 1);

        % Eslabones (Eslabón 1 rojo, Eslabón 2 azul)[cite: 4]
        plot(ax, [P0(1), P1(1)], [P0(2), P1(2)], '-o', 'LineWidth', 3, 'MarkerSize', 8, ...
             'Color', 'r', 'MarkerFaceColor', 'r'); % Eslabón 1 (Rojo)[cite: 4]
        plot(ax, [P1(1), P2(1)], [P1(2), P2(2)], '-o', 'LineWidth', 3, 'MarkerSize', 8, ...
             'Color', 'b', 'MarkerFaceColor', 'b'); % Eslabón 2 (Azul)[cite: 4]

        % Ajustes de escala gráfica
        grid(ax, 'on'); axis(ax, 'equal');
        xlabel(ax, 'X Global'); ylabel(ax, 'Y Global');
        legend(ax, {'Línea r', 'Eslabón 1 (l1=4)', 'Eslabón 2 (l2=2)'}, 'Location', 'northeast');
        hold(ax, 'off');
    end
end
