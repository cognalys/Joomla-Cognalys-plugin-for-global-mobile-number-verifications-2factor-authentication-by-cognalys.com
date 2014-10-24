<?php
/**
* Cognalys Plugin for Joomla 2.5
* @version $Id: cognalys.php $
* @package: Cognalys 
* ===================================================
* @author
* Name: Wuxing Quan
* Email: quanwx14@163.com
* Url: 
* ===================================================
* @copyright (C) 2014 Wuxing Quan. All rights reserved.
* @license see http://www.gnu.org/licenses/gpl-2.0.html  GNU/GPL.
* You can use, redistribute this file and/or modify
* it under the terms of the GNU General Public License as published by
* the Free Software Foundation.
*/
defined('_JEXEC') or die();

define('JOOMLA_COGNALYS_PLUGIN_VERSION', '1.0');

class plgSystemCognalys extends JPlugin
{
	var $enabledForms = array();
	var $addToForms =  array();
	
	/**
	 * Do something onAfterInitialise 
	 */
	function onAfterInitialise()
	{
		$this->cognalys_app_id = $this->params->get("cognalys_app_id");
		$this->cognalys_access_token = $this->params->get("cognalys_access_token");
		$isEnabledForForms = $this->getIsEnabledForForms();

		$check_cognalys = JRequest::getVar('check_cognalys');
		if ( $check_cognalys == 1 ) {
			$mobiles = JRequest::getVar('mobile');
			if ($mobiles != "") {
				$mobiles = split(',', $mobiles);
				$json = "";

				foreach($mobiles as $mobile) {
					$url = 'https://www.cognalys.com/api/v1/otp/?app_id=' . $this->cognalys_app_id . '&access_token=' . $this->cognalys_access_token . '&mobile=' . $mobile;
					if ($json != "") {
						$json .= ",";
					}
					$json .= file_get_contents($url);
				}
				$json = "[" . $json . "]";

				@header( 'Content-Type: application/json; charset=utf-8' );
				echo $json;
			}

			exit();
		}
		else if ( $check_cognalys == 2 ) {
			$keymatchs = JRequest::getVar('keymatch');
			$otps = JRequest::getVar('otp');
			if ($keymatchs != "") {
				$keymatchs = split(',', $keymatchs);
				$otps = split(',', $otps);
				$json = "";

				for ($i = 0; $i < count($keymatchs); $i ++) {
					$keymatch = $keymatchs[$i];
					$otp = $otps[$i];
					$url = 'https://www.cognalys.com/api/v1/otp/confirm/?app_id=' . $this->cognalys_app_id . '&access_token=' . $this->cognalys_access_token . '&keymatch=' . $keymatch . '&otp=' . $otp;
					if ($json != "") {
						$json .= ",";
					}
					$json .= file_get_contents($url);
				}
				$json = "[" . $json . "]";
				
				@header( 'Content-Type: application/json; charset=utf-8' );
				echo $json;
			}

			exit();
		}
		
		require_once(dirname(__FILE__).DIRECTORY_SEPARATOR.'coreForms.php');
	}

	private function isAdmin()
	{
		return preg_match("@".preg_quote(DIRECTORY_SEPARATOR)."administrator$@",JPATH_BASE);
	}

	function onAfterRoute()
	{
		$this->shouldInsertCognalys();
	}

	function skipCognalysInsertion($details, $formHTML)
	{
		$skipForm = false;
		if(isset($details['ignore_condition']) && 
			preg_match_all("/([^=&]+)=([^=&]*)/",$details['ignore_condition'],$ignore_condition_matches))
		{
			foreach($ignore_condition_matches[1] as $reqVarIndex=>$reqVar)
			{
				$regExp1 = "<input[^/>]+name=\"$reqVar\"[^/>]+/>";
				$regExp2 = "<input[^/>]+value=\"".$ignore_condition_matches[2][$reqVarIndex]."\"[^/>]+/>";
			
				if(preg_match("@".$regExp1."@",$formHTML,$match1) && preg_match("@".$regExp2."@",$match1[0],$match2))
				{
					$skipForm = true;
					break;
				}
			}
		}

		return $skipForm;		
	}

	function shouldInsertCognalys()
	{
		if($this->isAdmin())return false;
		$enabledForms = $this->enabledForms;
		
		$enableOnForm = true;
		
		foreach($enabledForms as $enabledForm => $details)
		{
			//echo $enabledForm ." : <br />";
			$enableOnForm = true;
			if($details['requestVars'] == '*')
			{
				$this->addToForms[] = $enabledForm;//add form for wild card,usually module forms
				JHtml::_('behavior.formvalidation');
				JHtml::_('formbehavior.chosen', 'select');	
				continue;
			}
			else
			{
				if(is_array($details['requestVars']))
				{	
					foreach($details['requestVars'] as $requestVar)
					{
						$continue = false;
						if( preg_match_all("/([^=&]+)=([^=&]+)/",$requestVar,$requestVar_matches)
							  &&
							 (JRequest::getVar('option') != $requestVar_matches[2][0]) 
							  )
						{
							//echo JRequest::getVar('option')." != ".$requestVar_matches[2][0]."<br />";
							$continue = true;
							break;
						}
					}
					if($continue) continue;
				}
				
				elseif( preg_match_all("/([^=&]+)=([^=&]+)/",$details['requestVars'],$requestVar_matches)
					  &&
					 (JRequest::getVar('option') != $requestVar_matches[2][0])
					 )
				{
					continue;
				}

				$enableOnForm = $this->confirmInsertCognalys($details);
				
				if($enableOnForm)
				{
					$this->addToForms[] = $enabledForm;
					JHtml::_('behavior.formvalidation');
					JHtml::_('formbehavior.chosen', 'select');	
				}
			}
		}
	}

	function confirmInsertCognalys($details)
	{
		$enableOnForm = false;
		$reqVars = array();
		if(!is_array($details['requestVars'])) {
			$reqVars[] = $details['requestVars'];
		}
		else {
			$reqVars = $details['requestVars'];
		}

		foreach($reqVars as $requestVarRegexp)
		{
			preg_match_all("/([^=&]+)=([^=&]+)/",$requestVarRegexp,$requestVar_matches);
			foreach($requestVar_matches[1] as $requestVarIndex => $requestVar)
			{
				$requestVarVals = preg_split("/\|/",$requestVar_matches[2][$requestVarIndex]);

				$enableOnForm = false;
				if(!in_array(trim(JRequest::getVar($requestVar)),$requestVarVals))
				{
					$enableOnForm = false;
					break;
				}
				else {
					$enableOnForm = true;
				}
			}
			if($enableOnForm) break;
		}
		return $enableOnForm;
	}

	function getIsEnabledForForms()
	{
		$plugin 	= JPluginHelper::getPlugin('system', 'osolcaptcha');
		$enabledForms = array(
								"enableForContactUs" , 
								"enableForComLogin", 
								"enableForRegistration",
								"enableForReset",
								"enableForRemind");
		$isEnabledForForm = array();
		foreach($enabledForms as $paramName )
		{
			$isEnabledForForm[$paramName] = ($this->params->get($paramName) == 'Yes');
		}
		return $isEnabledForForm;
	}

	function onBeforeRender()
	{
		JHtml::_('jquery.ui', array('core'));
		JHTML::stylesheet('plugins/system/cognalys/include/css/intlTelInput.css');
		JHTML::script('plugins/system/cognalys/include/js/intlTelInput.min.js');
		JHTML::stylesheet('plugins/system/cognalys/include/css/jquery-ui.min.css');
		//JHTML::stylesheet('plugins/system/cognalys/include/css/jquery.ui.dialog.min.css');
		JHTML::script('plugins/system/cognalys/include/js/jquery.ui.dialog.min.js');
		JHTML::stylesheet('plugins/system/cognalys/include/css/style.css');
		JHTML::script('plugins/system/cognalys/include/js/script.js');
	}

	public function onAfterRender()
	{
		$enabledForms = $this->enabledForms;//$this->getEnabledForms();
		$body = JResponse::getBody();
		
		foreach($this->addToForms as $addToForm)
		{
			//echo $addToForm."<br />";
			
			if(isset($enabledForms[$addToForm]['formId']))
			{
				$formId = $enabledForms[$addToForm]['formId'];
				$formRegExp = '<form[^>]+id="'.$formId.'".+</form>';
				$ajaxCheckFor = array('id' => $formId);
			}
			elseif(isset($enabledForms[$addToForm]['formName']))
			{
				$formId = $enabledForms[$addToForm]['formName'];
				$formRegExp = '<form[^>]+name="'.$formId.'".+</form>';
				$ajaxCheckFor = array('name' => $formId);
			}
			elseif(isset($enabledForms[$addToForm]['formaction_regExp']))
			{
				$returnVar = $this->getNonRefererableForm($enabledForms[$addToForm]);
				$formRegExp = '<form[^>]+action="[^\"]+'. $enabledForms[$addToForm]['formaction_regExp'].'.+".+</form>';
				$ajaxCheckFor = $returnVar['js_form_access'];//array('name' => $formId);
			}
			
			if(preg_match('@'.$formRegExp.'@isU', $body, $match_form) &&
						  !$this->skipCognalysInsertion($enabledForms[$addToForm],$match_form[0])
					)
			{
				$captchaHTML = '<input type=hidden name=cognalys_form_type value="' . $addToForm . '">';
				$checkContent = $enabledForms[$addToForm]['tagToPlaceCaptchaBefore'];
				//echo htmlspecialchars( $checkContent);
				$enhancedForm = str_replace($checkContent,$captchaHTML.$checkContent,$match_form[0]);
				$body = str_replace($match_form[0],$enhancedForm,$body);
			}
		}
		
		 // Set body
		JResponse::setBody($body);
		 
		return;
	}

	function createFormActionRegexp($regexp)
	{
		return preg_replace("/&/","(&amp;|&)",preg_quote(JRoute::_($regexp)));
	}

	function getNonRefererableForm($details)
	{
		
		$formRegExp = '<form[^>]+>.+</form>';
		$returnVar = array();
		
		$body = JResponse::getBody();
		if(preg_match_all('@'.$formRegExp.'@isU', $body, $match_forms))
		{
			foreach($match_forms[0] as $match_formIndex => $match_form)
			{
				if(isset($details['formaction_regExp']))
				{								
					$formActionRegExp =  "<form[^>]+action=\"([^\"]+)\"[^>]*>";
					//echo $match_form.htmlspecialchars($formActionRegExp)."<br />";
					if(preg_match("@".$formActionRegExp."@",$match_form,$match4) &&
								  preg_match("@".$details['formaction_regExp']."@",$match4[1],$match5)
								  )
					{
						
						$returnVar['js_form_access'] = array("action" => $match4[1]);
						return $returnVar;
					}
				}
				elseif(isset($details['no-id-form-ref-field']))
				{
					$formActionRegExp =  "<form[^>]+action=\"([^\"]+)\"[^>]*>";
					if(preg_match("@".$formActionRegExp."@",$match_form,$match4) &&
								  preg_match("@".$details['no-id-form-ref-field']."@",$match4[1],$match5)
								  )
					{
						//echo "@".$details['no-id-form-ref-field']."@";
						//echo htmlspecialchars($match4[1]).$match_form;
						$returnVar['js_form_access'] = array("action" => $match4[1]);
						return $returnVar;
					}
				}

				if(isset($details['no-id-form-ref-field']) && 
						preg_match_all("/([^=&]+)=([^=&]*)/",$details['no-id-form-ref-field'],$ignore_condition_matches)
						)
				{
					
					//echo $match_form;
					//echo "<pre>".print_r($ignore_condition_matches,true)."</pre>";
					foreach($ignore_condition_matches[1] as $reqVarIndex=>$reqVar)
					{
						$regExp1 = "<input[^/>]+name=\"$reqVar\"[^/>]+/>";
						$regExp2 = "<input[^/>]+value=\"".$ignore_condition_matches[2][$reqVarIndex]."\"[^/>]+/>";
						
						if(preg_match("@".$regExp1."@",$match_form,$match1) &&
									  preg_match("@".$regExp2."@",$match1[0],$match2))
						{
							$formActionRegExp =  "<form[^>]+action=\"([^\"]+)\"[^>]+>";
							if(preg_match("@".$formActionRegExp."@",$match_form,$match4))
							{
								//echo htmlspecialchars($match4[1]).$match_form;
								$returnVar['js_form_access'] = array("action" => $match4[1]);
							}
							
							return  $returnVar;
						}
					}
				}
			}
		}
			
		return false;
	}
}
?>