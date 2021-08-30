function [ pos, tmp_sc, tmp_rot, cscore,sscore] = tracking(  im,pos, p, polish ,frame)
%TRACKING Summary of this function goes here
%   Detailed explanation goes here
		%obtain a subwindow for detection at the position from last
    %frame, and convert to Fourier domain (its size is unchanged)
    %% find a proper window size
    large_num = 0;
    
    if polish > large_num
        % use smaller one to speedup
        w_sz0 = p.window_sz0;
        c_w = p.cos_window;
    else
        w_sz0 = p.window_sz_search0;
        c_w = p.cos_window_search;
    end
    
%         figure(5);
%         imagesc(im);
%         title('Video Frame');    
%     imwrite(frame2im(getframe(gcf)), ['E:\LDES-master\map_bolt\image\' num2str(frame) '.png']);
    %% translational estimating

    if p.isRotation
        patch = get_affine_subwindow(im, pos, p.sc,p.rot,w_sz0);
        patch0 = get_affine_subwindow(im, pos, p.sc,p.rot,round(w_sz0.*p.paddingplus));
    else
        sz_s = floor(p.sc.*w_sz0);
        patchO = get_subwindow(im, pos, sz_s);
        patch = mresize(patchO, w_sz0);
    end
    z = get_features(patch, p.features, p.cell_size, c_w);
    
%     [pcx,pcy]=oCenter(patch);
    
    zf = cellfun(@fft2, z, 'uniformoutput', false);
    ssz = cellfun(@size, zf, 'uniformoutput', false);
    
    %% calculate response of the classifier at all shifts
    wf = cellfun(@(imodel_xf,imodel_alphaf) bsxfun(@times, conj(imodel_xf), imodel_alphaf)/ numel(imodel_xf),...
        p.model_xf,p.model_alphaf, 'uniformoutput', false); 
    if polish <= large_num % get same size for filter and image
        w = cellfun(@(iwf, issz) padding(ifft2(iwf),[issz(1), issz(2)]), wf,ssz, 'uniformoutput', false);
        wf = cellfun(@fft2, w, 'uniformoutput', false);%,round(size(w)/2)
    end
    pad_sz = {};
    pad_sz{1} = [0,0];
    tmp_sz = ssz{1};
    
    % Compute convolution for each feature block in the Fourier domain
    % use general compute here for easy extension in future
    rff = cellfun(@(hf, xf, pad_sz) padarray(sum(bsxfun(@times, hf, xf), 3), pad_sz),...
         wf, zf, pad_sz, 'uniformoutput', false);
    rff = cellfun(@(rff) imresize(rff, tmp_sz(1:2), 'nearest'), rff, 'uniformoutput', false);
    rf = rff{1};
    response_cf = ifft2(rf,'symmetric');  %equation for fast detection
    
   %% color histogram map
    response_color = zeros(size(response_cf));
    if size(patch0,3) > 1
        object_likelihood = getColorSpace(patch0,p.features.pi,p.features.pl);
%         figure(2);
%         imagesc(object_likelihood);
%         title('Object-Likelihood Map');
%         imwrite(frame2im(getframe(gcf)), ['E:\LDES-master\map_skating2\Object-Likelihood Map\' num2str(frame) '.png']);

%         figure(4);
%         imshow(saliency);
%         title('Saliency Map');
%         imwrite(frame2im(getframe(gcf)), ['E:\LDES-master\map_skating2\Saliency Map\' num2str(frame) '.png']);
        if p.mbs==1          

            paramMBplus = getParam();
            paramMBplus.verbose = true; % to show run time for each step 
            object_likelihood = doMBS(object_likelihood, paramMBplus);
            
%             cos_window = hann(size(object_likelihood,1)) * hann(size(object_likelihood,2))';
%             object_likelihood=im2double(object_likelihood).*cos_window;

%             figure(3);
%             imshow(object_likelihood);   
%             title('Histogram-Saliency Map');
%             imwrite(frame2im(getframe(gcf)), ['E:\LDES-master\map_skating2\Histogram-Saliency Map\' num2str(frame) '.png']);
        end
        response_color = getCenterLikelihood(object_likelihood, p.target_sz0);
        response_color = mresize(response_color,size(response_cf));
    end
    response_cf = fftshift(response_cf);  

    %% combine the maps
%    p.merge_factor=0.35;
    response = (1 - p.merge_factor) *response_cf + p.merge_factor * response_color;
    %% sub-pixel search
    cscore =max(response(:));
    [pty, ptx] = find(response == cscore, 1);
    if p.isSubpixel
        slobe = 2;
        idy = pty-slobe:pty+slobe; idx = ptx-slobe:ptx+slobe;
        idx(idx>size(response,2))=size(response,2);idx(idx<1)=1;
        idy(idy>size(response,1))=size(response,1);idy(idy<1)=1;
        weightPatch = response(idy,idx);
        s = sum(weightPatch(:)) + eps;
        pty = sum(sum(weightPatch,2).*idy') / s;
        ptx = sum(sum(weightPatch,1).*idx) / s;
    end
    cscore = PSR(response,0.1);
    
    %% update the translational status
    vert_delta = pty - floor(size(response,1)/2);
    horiz_delta = ptx -floor(size(response,2)/2);
   
    local_trans = [vert_delta - 1, horiz_delta - 1];
    
%     local_trans = [vert_delta - 1, horiz_delta - 1]+[pcy,pcx];
    
    if p.isRotation
        sn = sin(p.rot); cs=cos(p.rot);
        pp = [p.sc(1)*cs,-p.sc(2)*sn;...
              p.sc(1)*sn, p.sc(2)*cs];
        pos = pos + (p.cell_size .* local_trans * pp);
    else
        pos = pos +  p.sc.*p.cell_size.*local_trans;
    end
    
    %% Estimating scale and rotation
    if p.isRotation
        patchL = get_affine_subwindow(im, pos, 1,p.rot,floor(p.sc.*p.scale_sz));
        %patchL = get_affine_subwindow(im, pos, 1,p.rot,floor(1.2.*p.sc.*p.scale_sz));
    else
        patchL = get_subwindow(im, pos, p.sc.*p.scale_sz);
    end
    
%     cos_window = hann(size(patchL,1)) * hann(size(patchL,2))';
%     patchL=im2double(patchL).*cos_window;
    
    
    patchL = mresize(patchL, p.scale_sz_window);
    % convert into logpolar
    patchLp = mpolar(double(patchL),p.mag);
    % get feature of scale and rotation
    patchLp= get_features_L(patchLp, p.features_scale, p.cell_size, []);
    
    [tmp_sc,tmp_rot,sscore] = estimateScale(p.modelPatch, patchLp ,p.mag);
    tmp_sc0=tmp_sc;
%     tmp_rot0=tmp_rot;

    if p.dc == 1
        %% length
        threshold=4;
        lengthL1=1:threshold;
        lengthL2=16-threshold:16+threshold;
        lengthL3=33-threshold:32;
        patchL_length = get_affine_subwindow(im, pos, 1,p.rot+tmp_rot,floor(p.sc.*p.scale_sz*1));
        patchL_length = mresize(patchL_length, p.scale_sz_window);

        % convert into logpolar
        patchLp_length = mpolar(double(patchL_length),p.mag);    

        % get feature of scale and rotation
        patchLp_length= get_features_L(patchLp_length, p.features_scale, p.cell_size, []);

        %segement mpolar pathch
        patchLp_length=patchLp_length([lengthL1 lengthL2 lengthL3],:,:);
        modelPatch=p.modelPatch([lengthL1 lengthL2 lengthL3],:,:);
        [tmp_sc_length,~,sscore_length] = estimateScale(modelPatch, patchLp_length ,p.mag);
        %% width
        widthL1=8-threshold:8+threshold;
        widthL2=24-threshold:24+threshold;
        patchL_width = get_affine_subwindow(im, pos, 1,p.rot+tmp_rot,floor(p.sc.*p.scale_sz*1));
        patchL_width = mresize(patchL_width, p.scale_sz_window);

        % convert into logpolar
        patchLp_width = mpolar(double(patchL_width),p.mag);    

        % get feature of scale and rotation
        patchLp_width= get_features_L(patchLp_width, p.features_scale, p.cell_size, []);

        %segement mpolar pathch
        patchLp_width=patchLp_width([widthL1 widthL2],:,:);
        modelPatch=p.modelPatch([widthL1 widthL2],:,:);
        [tmp_sc_width,~,sscore_width] = estimateScale(modelPatch, patchLp_width ,p.mag);
        if p.limit == 1
            if sscore_length > 1 * sscore && sscore_width > 1 * sscore 
               tmp_sc = [tmp_sc_width 0;0 tmp_sc_length]; 
            else
               tmp_sc = [tmp_sc0 0;0 tmp_sc0];
            end
        else
            tmp_sc = [tmp_sc_width 0;0 tmp_sc_length];     
        end
                % filter bad results
        if tmp_sc(1) > 1.4, tmp_sc(1) =1.4;end;
        if tmp_sc(4) > 1.4, tmp_sc(4) =1.4;end;
        if tmp_sc(1) < 0.6, tmp_sc(1) =0.6;end;   
        if tmp_sc(4) < 0.6, tmp_sc(4) =0.6;end;   
        if tmp_rot > 1, tmp_rot =0;end;
        if tmp_rot < -1, tmp_rot =0;end;
    
    else
            % filter bad results
        if tmp_sc > 1.4, tmp_sc =1.4;end;
        if tmp_sc < 0.6, tmp_sc =0.6;end;   
        if tmp_rot > 1, tmp_rot =0;end;
        if tmp_rot < -1, tmp_rot =0;end;    
    end

end

