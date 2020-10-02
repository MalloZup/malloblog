+++ 
date = 2020-10-02T20:08:03+02:00
title = "Saltstack standalone formula: resilency and zero error"
description = ""
slug = "" 
tags = []
categories = []
externalLink = ""
series = []
+++

# Saltstack Static types for standalone formula.

## Rationale:

Recently I wanted to run a formula saltstack as standalone. (without any master/minion) and locally.

Running a formula is quite complex if you don't know how to run it.

I do think that if you consider `zypper`, `salt-call` is the next layer up to zypper, in term of abstraction and context.

That is why i was motivated to create an simple CLI for my specific context; 

zypper is providing and ensuring a state written via spec, salt-call does something similar but on the more higher level and with systems configuration.

My approach isn't generic, since I focused on X formula,  it is more a pattern. ( a generic executor of formula, could be implemented via salt-call )


## Formula attributes on directory layout:

The formula states and pillar should be placed by the `rpm` in a well known place so the CLI can retrieve. So this problem is solved/delegated to zypper, by the RPM package of the formula.

( for details about this check out: https://github.com/SUSE/saphanabootstrap-formula/pull/105)

In addition, I researched how I could validate pillars with `golang` static types. 
The main problem was that pillar are jinja files, and they are changed dinamically via grains etc. For that see step 2).

This pattern works well if you have control of your formula so you can  add your types incrementally when you add new vars.. For custom upstream formula you don't control it might doesn't work.

# How the code works

The functionality of the code is following:

0) Pillars/States etc config is delegated to zypper.  (in my code they are hardcoded to /usr/share/salt-formulas etc.)

1) Create your structs golang from the pillar.yaml file
 Saltstack pillars are yaml  so you can autogenerate your structs easy.

2) Since Pillars can be jinja template, I struggled a bit to read this in golang.
To my rescue I found out that I can call salt util function and convert jinja to yaml, and salt does all the job for me for retrieving grains etc.

```
  // https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.slsutil.html#salt.modules.slsutil.renderer
        cmd := exec.Command("/usr/bin/salt-call", "--local", "slsutil.renderer", "default_renderer=jinja", formulaPillar)
```
Basically there I just say to salt-call: convert me the jinja grain, into yaml.
Since golang can read yaml with structs and create types, I can validate my pillars dinamically! ( it might seems obvious but is not)

3) Once I do have validate the pillars, I can run the formula with a salt-call state.apply call.


```
package main

import (
	"os"
	"os/exec"

	log "github.com/sirupsen/logrus"
	flag "github.com/spf13/pflag"
	"gopkg.in/yaml.v2"
)

type saphanaboostrapFormula struct {
	Local struct {
		Hana struct {
			InstallPackages   bool   `yaml:"install_packages"`
			SaptuneSolution   string `yaml:"saptune_solution"`
			SoftwarePath      string `yaml:"software_path"`
			HanaArchiveFile   string `yaml:"hana_archive_file"`
			HanaExtractDir    string `yaml:"hana_extract_dir"`
			SapcarExeFile     string `yaml:"sapcar_exe_file"`
			HaEnabled         bool   `yaml:"ha_enabled"`
			MonitoringEnabled bool   `yaml:"monitoring_enabled"`
			Nodes             []struct {
				Host     string `yaml:"host"`
				Sid      string `yaml:"sid"`
				Instance int    `yaml:"instance"`
				Password string `yaml:"password"`
				Install  struct {
					SoftwarePath       string `yaml:"software_path"`
					RootUser           string `yaml:"root_user"`
					RootPassword       string `yaml:"root_password"`
					HdbPwdFile         string `yaml:"hdb_pwd_file"`
					SystemUserPassword string `yaml:"system_user_password"`
					SapadmPassword     string `yaml:"sapadm_password"`
				} `yaml:"install,omitempty"`
				Primary struct {
					Name    string `yaml:"name"`
					Userkey struct {
						KeyName      string `yaml:"key_name"`
						Environment  string `yaml:"environment"`
						UserName     string `yaml:"user_name"`
						UserPassword string `yaml:"user_password"`
						Database     string `yaml:"database"`
					} `yaml:"userkey"`
					Backup struct {
						KeyName      string `yaml:"key_name"`
						UserName     string `yaml:"user_name"`
						UserPassword string `yaml:"user_password"`
						Database     string `yaml:"database"`
						File         string `yaml:"file"`
					} `yaml:"backup"`
				} `yaml:"primary,omitempty"`
				Exporter struct {
					ExpositionPort int    `yaml:"exposition_port"`
					MultiTenant    bool   `yaml:"multi_tenant"`
					User           string `yaml:"user"`
					Password       string `yaml:"password"`
					Timeout        int    `yaml:"timeout"`
				} `yaml:"exporter,omitempty"`
				SaptuneSolution string `yaml:"saptune_solution,omitempty"`
				Secondary       struct {
					Name            string `yaml:"name"`
					RemoteHost      string `yaml:"remote_host"`
					RemoteInstance  string `yaml:"remote_instance"`
					ReplicationMode string `yaml:"replication_mode"`
					OperationMode   string `yaml:"operation_mode"`
					PrimaryTimeout  int    `yaml:"primary_timeout"`
				} `yaml:"secondary,omitempty"`
				ScenarioType            string `yaml:"scenario_type,omitempty"`
				CostOptimizedParameters struct {
					GlobalAllocationLimit string `yaml:"global_allocation_limit"`
					PreloadColumnTables   bool   `yaml:"preload_column_tables"`
				} `yaml:"cost_optimized_parameters,omitempty"`
			} `yaml:"nodes"`
		} `yaml:"hana"`
	}
}

const (
	//	# these variables are formula specific:
	formulaLog  = "/var/log/salt-hana-formula.log"
	formulaName = "hana"
	// this is where the pillar are located
	formulaConfig = "/usr/share/salt-formulas/config/hana"
	formulaPillar = "/usr/share/salt-formulas/config/hana/pillar/hana/hana.sls"
)

var (
	// global --help flag
	helpFlag *bool
)

func validatePillar() {
	var c saphanaboostrapFormula
	// convert jinja to yaml
	// https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.slsutil.html#salt.modules.slsutil.renderer
	cmd := exec.Command("/usr/bin/salt-call", "--local", "slsutil.renderer", "default_renderer=jinja", formulaPillar)
	stdout, err := cmd.Output()
	if err != nil {
		log.Error(err)
	}

	log.Info("[PREFLIGHT]: pillar rendered... converting to yaml")
	err = yaml.Unmarshal(stdout, &c)
	if err != nil {
		log.Fatalf("Formula Pillar data is not valid!: %v", err)
	}
	log.Printf("--- t:\n%v\n\n", c)
	log.Info("[PREFLIGHT]: pillar valid!")
}

func main() {

	log.Infof("[PREFLIGHT]: validating pillar of salt formula %s", formulaName)

	// copy grains to config of formula
	_, err := exec.Command("/usr/bin/cp", "/etc/salt/grains", formulaConfig).Output()
	if err != nil {
		log.Errorf("error while rendindering pillar jinja file %s", err)
	}

	// render and validate via static types pillars
	validatePillar()

	// run formula
	log.Infof("[FORMULA]: formula %s will be executed. Please wait..", formulaName)
	formulaOut, err := exec.Command("/usr/bin/salt-call", "--local", "--log-level=info",
		"--retcode-passthrough", "--force-color", "--config="+formulaConfig, "state.apply", formulaName).CombinedOutput()

	if err != nil {
		log.Errorf("error while executing salt formula %s, %s", err, formulaOut)
	}

}
```
